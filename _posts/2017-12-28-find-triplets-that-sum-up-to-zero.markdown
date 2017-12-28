---
layout: post
title:  "Find triplets that sum up to zero"
tags: c++ algorithms
---

> Given an array, find all unique (no duplicates) triplets of numbers \\(a, b, c\\) that sum up to zero
>
> \\(a + b + c = 0\\)
>
> E.g.
>
>     {1, 4, -2, 1, 0, -1}
>
> the triplets are
>
>     {1, -2, 1}
>     {0, 1, -1}

The additional difficulty from a normal *sort-and-2sweep* after fixing each \\( a \\) element comes from the uniqueness of each triplet.

```
-6 -1 -1 -1 0 1 2
    |  |        |
    i  j        k
```

It is both necessary to skip \\( j \\) 's next elements after evaluating the solution `{-1, -1}` as well as \\( i \\)'s next elements if we've already pivoted a 2sweep with the same number.

{% highlight c++ %}
vector<vector<int>> find_unique_3sum(vector<int>& nums) {

    if (nums.size() < 3)
        return vector<vector<int>>();

    vector<vector<int>> res;

    sort(nums.begin(), nums.end());

    for(int i = 0; i < nums.size() - 2; ++i) {

        if (i > 0 && nums[i] == nums[i - 1]) continue; // Skip same result

        int j = i + 1;
        int k = nums.size() - 1;

        while(j < k) {

            int sum = nums[i] + nums[j] + nums[k];
            if (sum == 0) {
                res.push_back(vector<int>{nums[i], nums[j], nums[k]});

                ++j; --k;

                // Skip same result
                while(j < k && nums[j] == nums[j - 1]) ++j;
                while(j < k && nums[k] == nums[k + 1]) --k;

            } else {

                if (sum > 0)
                    --k;
                else
                    ++j;

            }
        }


    }

    return res;
}
{% endhighlight %}

The algorithms runs in \\( O(N^2) \\).
