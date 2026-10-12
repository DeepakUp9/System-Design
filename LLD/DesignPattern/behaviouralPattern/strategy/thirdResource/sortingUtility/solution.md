# Strategy Design Pattern – Sorting Utility (Simple Java)

## Question
Question:
 What is the Strategy Design Pattern, and how does it help in selecting algorithms at runtime?
Example Use-Case:
 In a sorting utility, you want to support multiple sorting strategies (bubble sort, quick sort, merge sort). 
 Using the Strategy Pattern, the client can choose the sorting algorithm dynamically based on dataset size or type.  
 
Strategy Design Pattern kya hai, aur ye runtime pe algorithm select karne mein kaise help karta hai?
Use-case: Sorting utility jahan bubble sort, quick sort, merge sort support karne hain aur client dataset ke size/type ke hisaab se algorithm choose kare.

---

## Strategy Pattern kya hai?
Ek **behavioral design pattern** jisme:
- Algorithms ki ek family define karte hain (bubble, quick, merge)
- Har algorithm ko **alag class** mein encapsulate karte hain
- Sabko **ek common interface** ke peeche rakhte hain, to runtime pe kabhi bhi swap kar sakte hain

## Runtime pe algorithm select kaise hota hai?
- Context class (`Sorter`) ke paas interface ka reference hota hai, concrete class ka nahi.
- Client `setStrategy()` se jo bhi strategy chahe de sakta hai.
- `Sorter` ko pata nahi hota kaunsa algorithm chal raha hai, wo bas `strategy.sort()` call karta hai.
- Isse `if/else` ya `switch` ki zaroorat khatam ho jati hai.

---

## Full Code (ek file, copy-paste karke run karo)

```java
import java.util.Arrays;

// 1. Strategy interface
interface SortStrategy {
    void sort(int[] arr);
}

// 2. Concrete strategies

// Bubble Sort: simple, chhote data ke liye theek. O(n^2)
class BubbleSort implements SortStrategy {
    public void sort(int[] arr) {
        int n = arr.length;
        for (int i = 0; i < n - 1; i++) {
            for (int j = 0; j < n - i - 1; j++) {
                if (arr[j] > arr[j + 1]) {
                    int temp = arr[j];
                    arr[j] = arr[j + 1];
                    arr[j + 1] = temp;
                }
            }
        }
    }
}

// Quick Sort: average O(n log n), in-place
class QuickSort implements SortStrategy {
    public void sort(int[] arr) {
        quickSort(arr, 0, arr.length - 1);
    }

    private void quickSort(int[] arr, int low, int high) {
        if (low < high) {
            int pivotIndex = partition(arr, low, high);
            quickSort(arr, low, pivotIndex - 1);
            quickSort(arr, pivotIndex + 1, high);
        }
    }

    private int partition(int[] arr, int low, int high) {
        int pivot = arr[high];
        int i = low - 1;
        for (int j = low; j < high; j++) {
            if (arr[j] < pivot) {
                i++;
                int temp = arr[i];
                arr[i] = arr[j];
                arr[j] = temp;
            }
        }
        int temp = arr[i + 1];
        arr[i + 1] = arr[high];
        arr[high] = temp;
        return i + 1;
    }
}

// Merge Sort: hamesha O(n log n), stable, extra space chahiye
class MergeSort implements SortStrategy {
    public void sort(int[] arr) {
        if (arr.length < 2) return;
        mergeSort(arr, 0, arr.length - 1);
    }

    private void mergeSort(int[] arr, int left, int right) {
        if (left < right) {
            int mid = (left + right) / 2;
            mergeSort(arr, left, mid);
            mergeSort(arr, mid + 1, right);
            merge(arr, left, mid, right);
        }
    }

    private void merge(int[] arr, int left, int mid, int right) {
        int[] temp = new int[right - left + 1];
        int i = left, j = mid + 1, k = 0;

        while (i <= mid && j <= right) {
            if (arr[i] <= arr[j]) temp[k++] = arr[i++];
            else temp[k++] = arr[j++];
        }
        while (i <= mid) temp[k++] = arr[i++];
        while (j <= right) temp[k++] = arr[j++];

        for (int x = 0; x < temp.length; x++) {
            arr[left + x] = temp[x];
        }
    }
}

// 3. Context: strategy ko use karta hai, concrete class ka pata nahi
class Sorter {
    private SortStrategy strategy;

    public Sorter(SortStrategy strategy) {
        this.strategy = strategy;
    }

    public void setStrategy(SortStrategy strategy) {   // runtime pe change
        this.strategy = strategy;
    }

    public void sort(int[] arr) {
        strategy.sort(arr);
    }
}

// 4. Client
public class Main {

    // Dataset size ke hisaab se strategy choose karna
    static SortStrategy chooseStrategy(int size) {
        if (size <= 10) {
            return new BubbleSort();      // chhota data
        } else if (size <= 1000) {
            return new QuickSort();       // medium data
        } else {
            return new MergeSort();       // bada data, stable + predictable
        }
    }

    public static void main(String[] args) {
        // (a) Manually strategy dena
        int[] data1 = {5, 3, 8, 1, 9, 2};
        Sorter sorter = new Sorter(new BubbleSort());
        sorter.sort(data1);
        System.out.println("Bubble : " + Arrays.toString(data1));

        // (b) Runtime pe strategy badalna (same Sorter object)
        int[] data2 = {40, 10, 30, 20, 50};
        sorter.setStrategy(new QuickSort());
        sorter.sort(data2);
        System.out.println("Quick  : " + Arrays.toString(data2));

        int[] data3 = {7, 2, 9, 4, 1};
        sorter.setStrategy(new MergeSort());
        sorter.sort(data3);
        System.out.println("Merge  : " + Arrays.toString(data3));

        // (c) Dataset size ke basis pe auto-select
        int[] data4 = {12, 4, 7, 1, 19, 3, 8, 15, 6, 2, 11, 5};   // size 12
        SortStrategy chosen = chooseStrategy(data4.length);
        sorter.setStrategy(chosen);
        sorter.sort(data4);
        System.out.println(chosen.getClass().getSimpleName() + ": " + Arrays.toString(data4));
    }
}
```

**Output**
```
Bubble : [1, 2, 3, 5, 8, 9]
Quick  : [10, 20, 30, 40, 50]
Merge  : [1, 2, 4, 7, 9]
QuickSort: [1, 2, 3, 4, 5, 6, 7, 8, 11, 12, 15, 19]
```

---

## Naya algorithm add karna (e.g. Insertion Sort)

```java
class InsertionSort implements SortStrategy {
    public void sort(int[] arr) {
        for (int i = 1; i < arr.length; i++) {
            int key = arr[i];
            int j = i - 1;
            while (j >= 0 && arr[j] > key) {
                arr[j + 1] = arr[j];
                j--;
            }
            arr[j + 1] = key;
        }
    }
}

// use:
sorter.setStrategy(new InsertionSort());
```

`Sorter` aur baaki algorithms ko touch nahi karna padta.

---

## Algorithms ka quick comparison

| Algorithm | Best | Average | Worst | Extra Space | Stable | Kab use karein |
|---|---|---|---|---|---|---|
| Bubble | O(n) | O(n²) | O(n²) | O(1) | Yes | Bahut chhota data, learning |
| Quick | O(n log n) | O(n log n) | O(n²) | O(log n) | No | General purpose, medium/large data |
| Merge | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes | Bada data, stable sort chahiye |

---

## Interview mein kya bolna hai

1. **Definition:** Strategy Pattern algorithms ki family ko alag classes mein encapsulate karta hai aur unhe interchangeable banata hai, runtime pe.
2. **Problem without it:** `sort(arr, "bubble")` jaisa method jisme `if/else` ho, to har naya algorithm add karne pe wahi method edit karna padta hai (Open/Closed Principle todta hai).
3. **Structure:**
   - **Strategy** (interface): `SortStrategy`
   - **Concrete Strategies:** `BubbleSort`, `QuickSort`, `MergeSort`
   - **Context:** `Sorter` (strategy ka reference rakhta hai)
   - **Client:** strategy choose karke context ko deta hai
4. **Benefits:**
   - Runtime pe algorithm change kar sakte hain
   - Naya algorithm = naya class, purana code safe
   - `if/else` hat jata hai
   - Har algorithm alag test ho sakta hai
5. **Drawbacks:**
   - Classes zyada ho jati hain
   - Client ko strategies ka pata hona chahiye (ya selection logic alag se likhna padta hai)
6. **Real Java example:** `Comparator` interface Strategy pattern hi hai.
   ```java
   list.sort((a, b) -> a.length() - b.length());     // length se sort
   list.sort(Comparator.reverseOrder());             // ulta sort
   ```
   `Collections.sort()` ek hi hai, par comparison ka tareeka (strategy) aap pass karte ho.
7. **Bonus point:** Real projects mein Java ka `Arrays.sort()` khud best algorithm choose karta hai, ye example sirf pattern samjhane ke liye hai.

## Strategy vs State (agar puche)
- **Strategy:** client strategy choose karta hai, strategies ek dusre ko nahi jaante.
- **State:** object khud apni state ke hisaab se behavior badalta hai, states ek dusre pe transition karti hain.

## One-liner
> Ek interface, har algorithm ka ek class, aur context bas `strategy.sort()` call karta hai. Algorithm badalna = strategy badalna.