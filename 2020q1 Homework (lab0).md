# 2020q1 Homework (lab0)
contributed by <` love1357983` >
###### tags: `linux2020`

## :penguin: 作業描述

[lab01](https://hackmd.io/@sysprog/linux2020-lab0#-作業要求)

## 開發紀錄

### queue_t

#### 說明：
為了讓 `q_insert_tail` 和 `q_size` 在 $O(1)$ 的時間複雜度，
在 `queue_t` 結構中增加以下欄位。
* `tail` 變數名稱 - `list_ele_t` 型態
* `size` 變數名稱 - `int` 型態

#### `queue.h` 實作:

```cpp
typedef struct {
    list_ele_t *head; 
    list_ele_t *tail;
    int size;
} queue_t;
```

### 建立

#### 說明：
在執行 `qtest` 程式一開始，會先呼叫 `queue.h` 中 `q_new` 函式，將欄位進行初始化。其中使用 `malloc` 函式，配置一個 `queue_t` 需要的空間，如果成功的話會回傳 `void*` 的記憶體位址；若失敗則回傳 `NULL` ，所以需要檢查 `malloc` 是否成功。

#### ***q_new***

```cpp
queue_t *q_new(void)
{
    queue_t *q = malloc(sizeof(queue_t));
    if (!q)
        return q;
    q->head = q->tail = NULL;
    q->size = 0;
    return q;
}

``` 

### 釋放

#### 說明：

將所建立 `queue_t` 型態的 `q` 記憶體空間釋放，包含每個節點上的動態配置的資料 `value` 欄位。在運行前一樣會先檢查 `queue_t` 是否存在，如果不存在直接返回。在這裡使用 `while` 迴圈走訪鏈結串列中各個節點。在 `while` 迴圈中，使用 `prev` 記錄目前 `tmp` 的記憶體位置，在 `tmp` 移動到下一個節點後，再將該節點中的 `next` 欄位作釋放動作，完成該節點在記憶裡空間完全釋放完畢。

#### ***q_free***

```cpp
void q_free(queue_t *q) 
{
    if (!q)
        return;
    list_ele_t *tmp = q->head;
    while (tmp) {
        list_ele_t *ptr = tmp;
        free((tmp->value));
        tmp = tmp->next;
        free(ptr);
    }
    free(q);
}
```

### `LIFO`插入

#### 說明：

使用 `LIFO` 方法進行排序，先建立 `newh` 的 `list_ele_t` 節點，將接收到的 `s` ，使用 `strdup` 複製字串加入 `newh` 的 `value` 中。把 `q->head` 接續在 `newh` 的 `next` 中，修改 `q->head` 為 `newh` ，判斷 `q_size` 是否為 `0` ，如果為 `0` 時，再把 `q->tail` 設定為 `newh` ，最後再將 `q_size` 增加 `1`。

#### ***q_insert_head***

```cpp
bool q_insert_head(queue_t *q, char *s)
{
    if (!q)
        return false;

    list_ele_t *newh = malloc(sizeof(list_ele_t));
    if (!newh) {
        free(newh);
        return false;
    }   

    newh->value = strdup(s);
    if (!newh->value) {
        free(newh);
        return false;
    }   
    newh->next = q->head;

    q->head = newh;
    if (!q_size(q))
        q->tail = newh;
    ++q->size;
    return true;
}
```

### `FIFO`插入

#### 說明：

同理，使用 `FIFO` 方法進行排序，先建立 `newt` 的 `list_ele_t` 節點，同樣將接收到的 `s` ，使用 `strdup` 複製字串加入 `newt` 的 `value` 中，把 `newt` 的 `next` 設定為 `NULL` ，把原 `q->tail->next` 為 `NULL` 設定為先前建立出的 `newt` ，再把原先的 `q->tail` 設定為 `newt` 進行取代，最後將 `q_size` 增加 `1` 。另外，在呼叫此函式開始時，會先取得 `q_size` 進行判斷，如果為 `0` 時，會呼叫與 `q_insert_head` 相同的程式碼，進行新增節點動作。

#### ***q_insert_tail***

```c=
bool q_insert_tail(queue_t *q, char *s)
{
    if (!q)
        return false;

    if (!q_size(q)) {
        q_insert_head(q, s);
        return true;
    }

    list_ele_t *newt = malloc(sizeof(list_ele_t));
    if (!newt) {
        free(newt);
        return false;
    }

    newt->value = strdup(s);
    if (!newt->value) {
        free(newt);
        return false;
    }
    newt->next = NULL;

    q->tail->next = newt;
    q->tail = newt;

    ++q->size;
    return true;
}
```

### 刪除

#### 說明：

將取得 `q->head->value` 節點內容複製到 `sp` 中，然後釋放該節點及資料所佔用的記憶體空間。先建立 `tmp` 的 `list_ele_t` 節點，記錄 `q->head` 該節點記憶體位置，再將 `q->head` 指向 `tmp->next` 記憶體位置，再把 `tmp` 記憶體位置進行釋放，將 `q_size` 減 `1` 。若 `q_size` 等於 `0` 時，須將 `q->tail` 記憶體空間進行清空動作。

#### ***q_remove_head***

```c=
bool q_remove_head(queue_t *q, char *sp, size_t bufsize)
{
    if (!q || !q->head)
        return false;

    if (bufsize > 0 && sp) {
        strncpy(sp, q->head->value, bufsize - 1);
        sp[bufsize - 1] = '\0';
    }

    list_ele_t *tmp = q->head;
    q->head = tmp->next;
    --q->size;
    if (!q_size(q))
        q->tail = NULL;
    free(tmp->value);
    free(tmp);
    return true;
}
```

### 個數

#### 說明：

如 `queue_t` 加入記錄佇列的 `size` 個數，可將計算 linked list 減少所需時間，將時間複雜度從 $O(n)$ 縮短到 $O(1)$ 。

#### ***q_size***

```c=
int q_size(queue_t *q)
{
    return (q == NULL) ? 0 : q->size;
}
```

### 反轉

#### 說明：

此函式限制在不建立額外配置空間下，先建立 `prev` 初始狀態為 `NULL`, `curr` 初始狀態為 `q->head`, `prec` 初始狀態為 `q->head->next` ，共三個 `list_ele_t` 節點。將當前的 `curr` 佇列位置從新指向 `next` 的 `prev` 記憶體位置，因為 `curr` 的 `next` 已經從新指向了，所以 `prec` 節點是為了讀取原有 `q->next` 佇列中的記憶體位置，使用 `while` 迴圈進行 `queue_t` 輪詢至 `prec` 為 `NULL` 為止，最後在將 `queue_t` 中 `head`, `tail` 進行對調動作。

#### ***q_reverse***

```cpp
void q_reverse(queue_t *q)
{
    if (!q || !q->head)
        return;

    list_ele_t *prev = NULL, *curr = q->head, *prec = q->head->next;

    while (prec) {
        curr->next = prev;
        prev = curr;
        curr = prec;
        prec = prec->next;
    }
    curr->next = prev;

    q->tail = q->head;
    q->head = curr;
    return;
}
```

### 排序

#### 說明：

以下程式碼使用 [Bubble Sort](https://en.wikipedia.org/wiki/Bubble_sort) 法，以==遞增順序==來排序鏈結串列的元素，其執行的時間複雜度為 $O(n^2)$ 。透過 `strcasecmp` 將 `first->value` 和 `curr->value` 比較，當`strcasecmp` 回傳數值小於 `0` 時，把兩個節點的 `value` 交換動作。

:::warning
:bookmark_tabs: 
這邊程式碼可能比較偷懶，只對 `list_ele_t` 中的 `value` 內容進行交換的動作，這邊有可能在兩者被分配的記憶體空間大小不同，而造成記憶體洩露 (memory leak)，後續會針對樣的問題進行實驗與研究。
:::

#### ***q_sort***

```cpp
void q_sort(queue_t *q)
{
    if (!q || !q->head)
        return;

    list_ele_t *first = q->head;
    while (first) {
        list_ele_t *curr = q->head;
        while (curr) {
            if (strcasecmp(first->value, curr->value) < 0) {
                char *tmp = curr->value;
                curr->value = first->value;
                first->value = tmp;
            }
            curr = curr->next;
        }
        first = first->next;
    }
    return;
}
```

## make test評分

// Mar 3 -> make test

```
---	Trace		Points
+++ TESTING trace trace-01-ops:
# Test of insert_head and remove_head
---	trace-01-ops	6/6
+++ TESTING trace trace-02-ops:
# Test of insert_head, insert_tail, and remove_head
---	trace-02-ops	6/6
+++ TESTING trace trace-03-ops:
# Test of insert_head, insert_tail, reverse, and remove_head
---	trace-03-ops	6/6
+++ TESTING trace trace-04-ops:
# Test of insert_head, insert_tail, size, and sort
---	trace-04-ops	6/6
+++ TESTING trace trace-05-ops:
# Test of insert_head, insert_tail, remove_head, reverse, size, and sort
# bear gerbil jiu kuo sheng
---	trace-05-ops	5/5
+++ TESTING trace trace-06-string:
# Test of truncated strings
---	trace-06-string	6/6
+++ TESTING trace trace-07-robust:
# Test operations on NULL queue
---	trace-07-robust	6/6
+++ TESTING trace trace-08-robust:
# Test operations on empty queue
---	trace-08-robust	6/6
+++ TESTING trace trace-09-robust:
# Test remove_head with NULL argument
---	trace-09-robust	6/6
+++ TESTING trace trace-10-malloc:
# Test of malloc failure on new
---	trace-10-malloc	6/6
+++ TESTING trace trace-11-malloc:
# Test of malloc failure on insert_head
---	trace-11-malloc	6/6
+++ TESTING trace trace-12-malloc:
# Test of malloc failure on insert_tail
---	trace-12-malloc	6/6
+++ TESTING trace trace-13-perf:
# Test performance of insert_tail
---	trace-13-perf	6/6
+++ TESTING trace trace-14-perf:
# Test performance of size
---	trace-14-perf	6/6
+++ TESTING trace trace-15-perf:
# Test performance of insert_tail, size, reverse, and sort
ERROR: Time limit exceeded.  Either you are in an infinite loop, or your code is too inefficient
ERROR: Not sorted in ascending order
---	trace-15-perf	0/6
+++ TESTING trace trace-16-perf:
# Test performance of sort with random and descending orders
# 10000: all correct sorting algorithms are expected pass
# 50000: sorting algorithms with O(n^2) time complexity are expected failed
# 100000: sorting algorithms with O(nlogn) time complexity are expected pass
ERROR: Time limit exceeded.  Either you are in an infinite loop, or your code is too inefficient
ERROR: Not sorted in ascending order
ERROR: Time limit exceeded.  Either you are in an infinite loop, or your code is too inefficient
ERROR: Not sorted in ascending order
ERROR: Time limit exceeded.  Either you are in an infinite loop, or your code is too inefficient
ERROR: Not sorted in ascending order
ERROR: Time limit exceeded.  Either you are in an infinite loop, or your code is too inefficient
ERROR: Not sorted in ascending order
---	trace-16-perf	0/6
+++ TESTING trace trace-17-complexity:
# Test if q_insert_tail and q_size is constant time complexity
Probably constant time
Probably constant time
---	trace-17-complexity	5/5
---	TOTAL		88/100
```