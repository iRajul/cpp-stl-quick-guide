### Binary Search 
### Problem 1 (Simple Version)
```cpp
int binary_search(int[] nums, int target) {
    int left = 0, right = nums.length - 1; 
    while(left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] < target) {
            left = mid + 1;
        } else if (nums[mid] > target) {
            right = mid - 1; 
        } else if(nums[mid] == target) {
            // Return directly
            return mid;
        }
    }
    // Return directly
    return -1;
}
```
### Left Bound (minimize mid for which condition remains true)
For such kind of problems where we need to minimize mid so as to satisfy a condition.
```cpp
int left_bound(int[] nums) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (condition(mid)) {
            right = mid - 1; //Shrink right side.
        } else 
            left  = mid + 1;
        }
    }
    return left;
}
```
### Problem 2.a
Find left most position of target in sorted array.
What should be condition here?
We need to find minimum `mid` position for which `arr[mid] >= target`.
so, simply apply above template.
```cpp
int left_bound(int[] nums) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] >= target) {
            right = mid -1; //Shrink right side.
        } else 
            left = mid + 1;
        }
    }
    if (left >= nums.size() || nums[left] != target)
	    return -1;
    return left;
}
```


### Problem 3 (Right Bound) (minimize mid for which condition remains true)
```cpp
int right_bound(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (condition(mid)) {
            left  = mid + 1; // Shrink left side
        } else {
            right = mid - 1;
        }
    }
    return right;
}
```

### Problem 3.a
Find right most position of target in sorted array.
What should be condition here?
We need to find maximum `mid` position for which `arr[mid] <= target`.
so, simply apply above template.
```cpp
int right_bound(int[] nums) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] <= target) {
            left  = mid + 1; //Shrink left side.
        } else {
            right = mid - 1;
        }
    }
    if (right < 0 || nums[right] != target)
	    return -1;
    return right;
}
```

Reference : 
https://labuladong.gitbook.io/algo-en/iii.-algorithmic-thinking/detailedbinarysearch
https://leetcode.com/problems/koko-eating-bananas/discuss/769702/Python-Clear-explanation-Powerful-Ultimate-Binary-Search-Template.-Solved-many-problems
