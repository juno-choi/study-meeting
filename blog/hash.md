`출처` [DEV_제임스 [자료구조] 해시(HASH) 알아보기](https://kang-james.tistory.com/entry/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0-%ED%95%B4%EC%8B%9CHASH-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0)

# 🔴 Hash 자료구조

해시는 저장 또는 검색에서 자주 활용되는 자료구조이다. 그 이유는 검색시 O(1)로 가장 빠르게 찾아낼 수 있는 자료구조이기 때문이다.

## 🟠 Hash 란

입력 데이터를 고정된 길이의 데이터로 변환된 값을 말한다. 다른 말로는 hash value, hash code, checksum 이라고 한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbcluEM%2FbtryF769jdT%2FtldrkmqK9arIia8rGKH5e0%2Fimg.png)

key값을 해시 함수(hash function)를 통해 해시 값으로 변환한다. 이렇게 변환된 값은 배열의 인덱스, 위치, 데이터 값을 저장하거나 검색할 때 활용된다.

### 🟢 자료구조 특징

- key에 value를 매핑하는 데이터 구조
- 해시 함수를 통해 키의 데이터를 배열에 저장할 주소 값을 계산
- 키를 통해 저장된 데이터를 빠르게 찾기 때문에, 저장 및 탐색 속도가 빠름


### 🟢 해시 함수 (Hash Function)

입력받은 데이터를 해시 값으로 변환해주는 알고리즘을 말한다. 

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

java의 hash 함수는 다음과 같다.

### 🟢 해시 테이블 (Hash Table)

key와 value를 함께 저장해둔 데이터 구조이다. 데이터가 행과 열로 구성된 표에 저장되는 것과 유사하다.

테이블에 데이터를 저장할 때 위치는 무작위로 지정되어 작성된다. 즉, 중간에 비어있는 공간이 있을 수도 있다.

`버킷 (Bucket)` 하나의 주소를 갖는 파일의 한 구역, 커밋의 크기는 같은 주소에 포함될 수 있는 레코드의 수

`슬롯 (Slot)` 한 개의 레코드를 저장할 수 있는 공간, 한 버킷엔 여러 개의 슬롯이 있음

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FS2WQc%2FbtryFn3yOlq%2FDEDRa7Ty65tjYqej9ax51k%2Fimg.png)


java에서는 `new HashMap<>()` 으로 hashMap을 생성했을 때 최초의 사이즈는 `DEFAULT_INITIAL_CAPACITY` 값으로 16의 값을 가진다. 그리고 `load_factor`의 값은 `DEFAULT_LOAD_FACTOR`로 0.75 값을 가지고 있는데

이는 hashMap을 최초로 생성했을 때 테이블의 크기를 16으로 만들고 0.75를 곱한 값인 12까지는 저장하고 13에서 리사이징이 일어난다는 의미이다.

리사이징이 자주 일어나면 해싱을 모두 다시 해야되기 때문에 O(N)의 시간복잡도가 실행된다. 최적화를 위해서는 일정한 크기의 데이터라면 해당 데이터의 크기를 미리 초기화 해두는 것이 최적화에 더욱 도움이 된다.

### 🟢 해싱 (Hashing)

해시 함수를 통해 해시 값을 얻고 해시 테이블에 저장하는 과정까지의 행위를 말한다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fb8A2pM%2FbtryI5m2piK%2Fcv3C7kfDq5xsaQuv4inKWK%2Fimg.png)

다음 그림과 같은 구조를 가지는 것이다.

### 해시 충돌(Collision)

해시 함수를 통해 해시 값을 만들고 해시 테이블에 저장하는 과정에서 계산 결과가 같아 값이 중복되는 경우를 해시 충돌이라고 한다.

`Chaning 기법`
- 개방 해싱 또는 Open Hashing 기법
- 해시 테이블 저장공간 외의 공간을 활용하는 기법
- 충돌이 발생했을 때, Linked List 자료구조를 사용해서 해결하는 방법
- java hashMap의 구조

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FwBocM%2FbtryJjFpTos%2FHM3Ap9sSUwDGxT6KawqHJ0%2Fimg.png)

`Linear Probing 기법`
- 폐쇄 해싱 또는 Open Address 기법
- 충돌이 발생했을 때, 해당 해시 주소(index) 다음 주소부터 맨 처음까지 순회하며 빈 공간을 찾는 방식
- 저장 공간 활용도를 높이기 위한 기법

### 🟢 java의 해시 충돌

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {
            for (int binCount = 0; ; ++binCount) {
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

hashMap에서 put에서 적용되는 putVal()의 메서드 내부이다. 이중에서

```java
for (int binCount = 0; ; ++binCount) {
    if ((e = p.next) == null) {
        p.next = newNode(hash, key, value, null);
        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
            treeifyBin(tab, hash);
        break;
    }
    if (e.hash == hash &&
        ((k = e.key) == key || (key != null && key.equals(k))))
        break;
    p = e;
}
```

hash의 충돌이 발생했을 때, 새로운 노드를 만들어 newNode를 통해 노드를 추가해주고 만일 노드의 수가 `TREEIFY_THRESHOLD` 값 이상일 경우에는 `treeifyBin()`을 통해 linked list 구조에서 tree 구조로 변경하는 것을 의미한다.

`TREEIFY_THRESHOLD` 값은 java에서는 8로 정의되어 있다. 즉 하나의 bucket에서 8개 이상의 노드가 있을 경우에 treeifyBin()을 호출한다.

```java
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        TreeNode<K,V> hd = null, tl = null;
        do {
            TreeNode<K,V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
```

내부적으로 총 map의 크기가 `MIN_TREEIFY_CAPACITY` (크기는 64이다.) 보다 작을 경우에는 해시를 새롭게 만들어 키값을 새롭게 정렬하여 충돌을 다시 피해갈 수 있도록 조율한다. 하지만 `MIN_TREEIFY_CAPACITY` 값보다 큰 경우에는 tree 구조로 변경하도록 한다.

여기서 특정 크기가 되었을 때 기존 linked list 구조에서 tree 구조로 변경하는 이유는 linked list의 형태의 경우 탐색에서 O(n)의 시간 복잡도를 가지게 된다. 

tree 구조로 변경할 경우 탐색에서 O(logn)의 시간복잡도를 가지기에 특정 크기 이상이 되었을 땐 tree 구조로 변경하여 성능에서 이점을 가져오려고 하는 부분이다.

### 🟢 해시의 장점과 단점

`장점`

- 데이터 저장/읽기 속도가 빠름 O(1)
- 키에 대한 데이터가 있는지 체크 여부가 쉬움

`단점`

- 일반적으로 메모리 크기가 더 많이 사용 됨
- 해시 충돌을 해결하기 위해 별도의 자료구조 필요
- 최악의 경우 O(n) or O(logn)의 시간 복잡도를 가짐