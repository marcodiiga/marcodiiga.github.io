---
layout: post
title:  "Multithreaded code wrapping"
tags: algorithms multithreading concurrency
---

Wrapping a textline in a code editor involves calculating a suitable splitting point to break down a line into multiple sublines.
Given a physical line \\( p \\), the task needs to determine one or more splitting points to generate one or multiple editor lines (or virtual lines) \\( e_i \\).

{% highlight c++ %}
struct EditorLine {
  EditorLine(std::string str);
  std::vector<char> m_characters;
};

struct PhysicalLine {
  PhysicalLine(EditorLine&& editorLine) {
    m_editorLines.emplace_back(std::forward<EditorLine>(editorLine));
  }
  PhysicalLine(const PhysicalLine&) = default;
  PhysicalLine() = default;
  std::vector<EditorLine> m_editorLines;
};
{% endhighlight %}

Finding the multivalued function \\( f(p) \\) and computing these points efficiently is a nontrivial task.

A straightforward way of doing this is as follows

{% highlight c++ %}
std::vector<PhysicalLine> phLineVec;
{
  std::string restOfLine;
  std::vector<EditorLine> edLines;
  edLines.reserve(10); // Should be enough for every splitting
  if (line.size() * m_codeView.getCharacterWidthPixels() > m_wrapWidth) { // Wrap
    edLines.clear();
    restOfLine = line;

    // Calculate the allowed number of characters per editor line
    int maxChars = static_cast<int>(m_wrapWidth / m_codeView.getCharacterWidthPixels());
    if (maxChars < 10)
      maxChars = 10; // Keep it to a minimum

    // Start the wrap-splitting algorithm or resort to a brute-force character splitting one if
    // no suitable spaces could be found to split the line
    while (restOfLine.size() > maxChars) {

      int bestSplittingPointFound = -1;
      for (int i = 0; i < restOfLine.size(); ++i) {
        if (i > maxChars)
          break; // We couldn't find a suitable space split point for restOfLine
        if (restOfLine[i] == ' ' && i != 0 /* Doesn't make sense to split at 0 pos */)
          bestSplittingPointFound = i;
      }

      if (bestSplittingPointFound != -1) { // We found a suitable space point to split this line
        edLines.push_back(restOfLine.substr(0, bestSplittingPointFound));
        restOfLine = restOfLine.substr(bestSplittingPointFound);
      } else {
        // No space found, brutally split characters (last resort)
        edLines.push_back(restOfLine.substr(0, maxChars));
        restOfLine = restOfLine.substr(maxChars);
      }
    }
    edLines.push_back(restOfLine); // Insert the last part and proceed

    std::vector<EditorLine> edVector;
    std::copy(edLines.begin(), edLines.end(), std::back_inserter(edVector));
    phLineVec.resize(1);
    phLineVec[0].m_editorLines = std::move(edVector);

  }
  else { // No wrap or the line fits perfectly within the wrap limits

    EditorLine el(line);
    phLineVec.emplace_back(std::move(el));
  }
}
{% endhighlight %}

Cycling the code above per each physical line of a document will soon degrade performances and cause scalability problems when loading huge code documents or rewrapping/relexing on the fly.

Map and reduce
--------------

A [*map and reduce*](https://en.wikipedia.org/wiki/MapReduce) approach can be used to exploit thread-level parallelism. Let us define a map function \\( m(x) \\) and a reduce function \\( r(x, y) \\). For the task at hand a blocking ordered map-reduce
operation can be defined as

$$ 
\begin{align}
& m(x) \Rightarrow \text{find the splitting points and return a vector of editor lines} \\
& r(x,y) \Rightarrow \text{join the vector of editor lines to the global result vector of editor lines}
\end{align}
$$

Similarly for the naive method above, we can now define a multithreaded version for map and reduce routines

{% highlight c++ %}
std::function<std::vector<PhysicalLine>(const std::string&)> mapFn = [&, this](const std::string& line) {

  std::vector<PhysicalLine> phLineVec;
  std::string restOfLine;
  std::vector<EditorLine> edLines;
  edLines.reserve(10);
  if (line.size() * m_codeView.getCharacterWidthPixels() > m_wrapWidth) {

    edLines.clear();
    restOfLine = line;

    int maxChars = static_cast<int>(m_wrapWidth / m_codeView.getCharacterWidthPixels());
    if (maxChars < 10)
      maxChars = 10;

    while (restOfLine.size() > maxChars) {

      int bestSplittingPointFound = -1;
      for (int i = 0; i < restOfLine.size(); ++i) {
        if (i > maxChars)
          break; // We couldn't find a suitable space split point for restOfLine
        if (restOfLine[i] == ' ' && i != 0 /* Doesn't make sense to split at 0 pos */)
          bestSplittingPointFound = i;
      }

      if (bestSplittingPointFound != -1) { // We found a suitable space point to split this line
        edLines.push_back(restOfLine.substr(0, bestSplittingPointFound));
        restOfLine = restOfLine.substr(bestSplittingPointFound);
      } else {
        // No space found, brutally split characters (last resort)
        edLines.push_back(restOfLine.substr(0, maxChars));
        restOfLine = restOfLine.substr(maxChars);
      }
    }
    edLines.push_back(restOfLine); // Insert the last part and proceed

    std::vector<EditorLine> edVector;
    std::copy(edLines.begin(), edLines.end(), std::back_inserter(edVector));
    phLineVec.resize(1);
    phLineVec[0].m_editorLines = std::move(edVector);

  } else { // No wrap or the line fits perfectly within the wrap limits

    EditorLine el(line);
    phLineVec.emplace_back(std::move(el));
  }
  return phLineVec;
};


std::function<void(std::vector<PhysicalLine>&, const std::vector<PhysicalLine>&)> reduceFn =
         [&, this](std::vector<PhysicalLine>& accumulator, const std::vector<PhysicalLine>& pl) {

  accumulator.insert(accumulator.end(), pl.begin(), pl.end());

  m_numberOfEditorLines += static_cast<int>(pl[0].m_editorLines.size()); // Some more EditorLine
  std::for_each(pl[0].m_editorLines.begin(), pl[0].m_editorLines.end(), [this](const EditorLine& eline) {
    int lineLength = static_cast<int>(eline.m_characters.size());
    if (lineLength > m_maximumCharactersLine) // Check if this is the longest line found ever
      m_maximumCharactersLine = lineLength;
  });

};
{% endhighlight %}

Exploiting the C++ standard library integrated support for native threads and metaprogramming introspection to ensure arity and type correctness among the mapping and reduce routines, a blocking
ordered map-reduce can be written as follows

{% highlight c++ %}
namespace {

  template <typename T>
  struct prototype_traits;

  template <typename Ret, typename... Args>
  struct prototype_traits < std::function<Ret(Args...)> > :
    public prototype_traits<Ret(Args...)>
  {};

  template <typename Ret, typename... Args>
  struct prototype_traits<Ret(Args...)> {

    constexpr static size_t arity = sizeof...(Args);

    using return_type = Ret;

    template <size_t i>
    struct param_type {
      using type = typename std::tuple_element<i, std::tuple<Args...>>::type;        
    };
  };
}

template <
  typename Accumulator,
  typename Sequence,
  typename Map,
  typename Reduce
>
Accumulator blockingOrderedMapReduce(const Sequence& sequence, const Map& map, 
                                     const Reduce& reduce, const size_t maps_per_threads_hint = 20U) 
{
  static_assert(
    prototype_traits<Map>::arity == 1 &&
    std::is_same<
      typename std::remove_reference<typename prototype_traits<Map>::return_type>::type,
      Accumulator
    >::value &&
      (std::is_same<
        typename std::remove_cv<typename std::remove_reference<typename prototype_traits<Map>::template param_type<0>::type>::type>::type,
        typename Sequence::value_type
      >::value || std::is_same<
        typename std::remove_reference<typename prototype_traits<Map>::template param_type<0>::type>::type,
        typename Sequence::value_type
      >::value),
    "Map intermediate / input type doesn't match the expected accumulator / input value"
    );

  static_assert(
    prototype_traits<Reduce>::arity == 2 &&
    std::is_same<
      typename std::remove_reference<typename prototype_traits<Reduce>::template param_type<0>::type>::type,
      Accumulator
    >::value &&
      (std::is_same<
        typename std::remove_cv<typename std::remove_reference<typename prototype_traits<Reduce>::template param_type<1>::type>::type>::type,
        Accumulator
      >::value || std::is_same<
        typename std::remove_reference<typename prototype_traits<Reduce>::template param_type<1>::type>::type,
        Accumulator
      >::value),
    "Reduce parameters don't match / incompatible with accumulator type"
    );
 
  using SequenceIterator = typename Sequence::const_iterator;
  static_assert(
    std::is_same<
      typename std::iterator_traits<SequenceIterator>::iterator_category,
      std::random_access_iterator_tag
    >::value,
    "Sequence hasn't random access iterators"
    );
  using AccumulatorIterator = typename Sequence::const_iterator;
  static_assert(
    std::is_same<
      typename std::iterator_traits<AccumulatorIterator>::iterator_category,
      std::random_access_iterator_tag
    >::value,
    "Accumulator hasn't random access iterators"
    );

  if (sequence.size() == 0)
    return Accumulator{};

  auto numThreads = std::max(1U, std::thread::hardware_concurrency());
  size_t minMapsPerThread = maps_per_threads_hint;

  size_t mapsPerThread;
  while(true) {
    mapsPerThread = static_cast<size_t>(
      std::ceil(sequence.size() / static_cast<double>(numThreads))
      );
    if (mapsPerThread < minMapsPerThread) {
      numThreads = std::max(numThreads / 2, 1U);
      if (numThreads > 1)
        continue;
    }
    break;
  }

  std::mutex barrier_mutex;

  Accumulator result;
  std::map<size_t, Accumulator> threads_partials_map;
  std::vector<std::promise<void>> threads_promises;
  std::vector<std::future<void>> threads_futures;

  auto threadDispatcher = [&](size_t threadId, size_t start, size_t end) 
  {
    typename prototype_traits<Map>::return_type thread_partials;
    for (size_t i = start; i < end; ++i) {
      auto intermediate = map(sequence[i]);
      reduce(thread_partials, intermediate);
    }

    // ~-~-~-~-~-~-~-~-~ Sync barrier ~-~-~-~-~-~-~-~-~
    {
      std::unique_lock<std::mutex> lock(barrier_mutex);
      threads_partials_map.emplace(threadId, std::move(thread_partials));
      threads_promises[threadId].set_value();
    }      
  };

  std::vector<std::thread> threads;
  for (size_t i = 0; i < numThreads; ++i) {
    size_t start = i * mapsPerThread;
    size_t end;
    if (numThreads == 1)
      end = sequence.size();
    else
      end = std::min(sequence.size(), start + mapsPerThread);
    {
      std::unique_lock<std::mutex> lock(barrier_mutex);
      threads_promises.emplace_back();
      threads_futures.emplace_back(threads_promises.back().get_future());
      threads.emplace_back(threadDispatcher, i, start, end);
    }
  }

  for (auto& future : threads_futures)
    future.wait();

  for (size_t i = 0; i < numThreads; ++i) {
    std::copy(threads_partials_map[i].begin(), threads_partials_map[i].end(),
              std::back_inserter(result)); 
  }

  for (auto& thread : threads)
    thread.join();

  return result;
}
{% endhighlight %}


Dynamic tiling with threadpool support
--------------------------------------

Although the map and reduce approach should prove considerably faster with respect to the naive single-core approach, thread spawning (especially in OSes designed to carry and initialize a non-trivial amount of thread data) might prevent additional performances squeezing. Furthermore having an inter-thread cooperation, as is the case when rendering with the data from a style database gained from a lexing phase, will render the map and reduce approach even harder to implement properly.

We can alleviate the first problem by using a simple threadpool

{% highlight c++ %}
class ThreadPool {
public:
  ThreadPool(size_t N_Threads) : m_NThreads(N_Threads) {
    m_workloadReady.resize(m_NThreads, false);
    for(auto i = 0; i < m_NThreads; ++i)
      m_threads.emplace_back(&ThreadPool::threadMain, this, i);
  }
  ~ThreadPool() {
    m_sigterm = true;
    m_cv.notify_all();
    for (auto& thread : m_threads) {
      if (thread.joinable())
        thread.join();
    }
  }
  void setCallback(std::function<void(size_t)> callback) {
    m_callback = callback;
  }
  void dispatch() {
    std::fill(m_workloadReady.begin(), m_workloadReady.end(), true);
    m_workloadReady.resize(m_NThreads, true);
    m_cv.notify_all();
  }

  const size_t m_NThreads;

private:
  void threadMain(size_t threadIdx) {
    while (!m_sigterm) {
      {
        std::unique_lock<std::mutex> lock(m_mutex);
        while (!m_sigterm && !m_workloadReady[threadIdx]) {
          m_cv.wait(lock);
        }
      }
      if (m_sigterm)
        return;

      m_callback(threadIdx);

      {
        std::unique_lock<std::mutex> lock(m_mutex);
        m_workloadReady[threadIdx] = false;
      }
    }
  }

  std::vector<std::thread> m_threads;
  bool m_sigterm = false;
  std::vector<bool> m_workloadReady;
  std::mutex m_mutex;
  std::condition_variable m_cv;
  std::function<void(size_t)> m_callback;
};
{% endhighlight %}

The line splitting algorithm will remain mostly the same as in the map and reduce approach.
Anyway the latter issue needs a complete algorithm restructuring which assigns a fixed amount of workload by tiling the input

{% highlight c++ %}
  // Subdivide the document's lines into a suitable amount of workload per thread
  size_t numThreads = m_codeView.m_threadPool.m_NThreads;
  size_t minLinesPerThread = 20u;

  while (true) {
    m_linesPerThread = static_cast<size_t>(
      std::ceil(m_plainTextLines.size() / static_cast<float>(numThreads))
      );
    if (m_linesPerThread < minLinesPerThread) {
      numThreads /= 2;
      if (numThreads < 1)
        numThreads = 1;
      else if (numThreads > 1)
        continue;
    }
    break;
  }
{% endhighlight %}

Now a thread-local rendering can be performed on the chunked input  

{% highlight c++ %}
m_threadPool.setCallback(std::bind(&Document::threadProcessChunk, this, 
                                   std::placeholders::_1));
m_threadPool.dispatch();
{% endhighlight %}

The main `threadProcessChunk()` algorithm follows the pseudocode

    find previous or current style segment info (from lexing phase)
    calculate next position to reach according to the minumum between segment end / line end
    draw the text (use acceleration structures to gain insights on the style to be used)

The complete algorithm is quite lengthy and can be found in the [Varco repository](https://github.com/VarcoDevs/Varco).

Tallying up
-----------

The dynamic tiling algorithm with threadpool support proved to be around **10X** faster on the target architecture by providing the
following average results

    Average complete document wrap/lex recalculation and rendering
    map and reduce:                 avg 85.487 ms
    dynamic tiling with threadpool: avg 8.566 ms

The final result is rendered in the following video

<center><iframe width="560" height="315" src="https://www.youtube.com/embed/GUp48wzxy9o" frameborder="0" allowfullscreen></iframe></center>