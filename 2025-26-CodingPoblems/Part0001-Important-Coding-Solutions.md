# Java Coding Problems - 2025-2026

##### 1) Java program to check if two strings are anagrams
##### 2) Java program that groups anagrams together
##### 3) Java program to LRU Implementation









##### Ans 1) Java program to check if two strings are anagrams

```java
public class P1 {

	public static void main(String[] args) {

		String str1 = "Listen";
		String str2 = "Silent";

		str1 = str1.replaceAll("\\S", "").toLowerCase();
		str2 = str2.replaceAll("\\S", "").toLowerCase();

		char[] a1 = str1.toCharArray();
		char[] a2 = str2.toCharArray();

		Arrays.sort(a1);
		Arrays.sort(a2);

		if (Arrays.equals(a1, a2)) {
			System.out.println("Anagrams");
		} else {
			System.out.println("Not Anagrams");
		}

	}

}
```
* Ref and check other techniques - https://chatgpt.com/c/68bfa20e-bc80-8329-bd63-164cc8dd14e0

##### Ans 2) Java program that groups anagrams together

```java
public class AnagramGroups {
    public static void main(String[] args) {
        String[] words = {"eat", "tea", "ate", "bat", "tab", "tan", "nat"};

        Map<String, List<String>> map = new HashMap<>();

        for (String word : words) {
            // Sort characters in word → this will be the key
            char[] chars = word.toCharArray();
            Arrays.sort(chars);
            String key = new String(chars);

            // Add word to the correct group
            map.computeIfAbsent(key, k -> new ArrayList<>()).add(word);
        }

        // Print groups
        for (List<String> group : map.values()) {
            System.out.println(group);
        }
    }
}
```

<img width="838" height="668" alt="image" src="https://github.com/user-attachments/assets/a161b1f0-0b91-4e38-9ffb-fc6740b6f3e6" />
* For mor details and diff waus -> https://chatgpt.com/c/68bfaa26-b8e8-8326-914c-33faca7ca730

##### 3) Java program to LRU Implementation

<img width="792" height="737" alt="image" src="https://github.com/user-attachments/assets/791a0b15-0ff5-40b6-b795-f6ce629b2dba" />

```java
package com.vkakarla.imp;

import java.util.LinkedHashMap;
import java.util.Map;

public class LRUCache<K, V> extends LinkedHashMap<K,V>{
	
	private final int capacity;
	
	public LRUCache(int capacity) {
		super(capacity, 0.75f, true);
		this.capacity=capacity;
	}
	
	protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
		return size() > capacity;
	}
	
	public static void main(String[] args) {
       
		LRUCache<Integer, String> cache = new LRUCache<>(3);
		cache.put(1, "one");
		cache.put(2, "two");
		cache.put(3, "three");
		cache.put(4, "four");
		cache.get(3);         // moves 3 to recent
        cache.get(2);         // moves 2 to recent
        cache.put(1, "one");  // removes 4
        cache.get(3);         // moves 3 to recent
        cache.get(1);         // moves 1 to recent
			
		cache.forEach((K, V) -> System.out.println(K + "-->" + V));
	}

}

Out Put: 
2-->two
3-->three
1-->one
```
<img width="868" height="574" alt="image" src="https://github.com/user-attachments/assets/567e7406-6924-43a2-b16d-4bb0d5077734" />
<img width="808" height="741" alt="image" src="https://github.com/user-attachments/assets/32f58986-e05e-46df-8f5f-abf2a08feea2" />
<img width="819" height="642" alt="image" src="https://github.com/user-attachments/assets/7848904b-6148-450b-9626-c302af95181e" />
<img width="809" height="738" alt="image" src="https://github.com/user-attachments/assets/79f13e92-0cd4-45fa-b913-467b5cc72a36" />
<img width="821" height="476" alt="image" src="https://github.com/user-attachments/assets/cd10f476-3dc5-4d2a-8d0f-c22199345e84" />


* **also show a thread-safe version (with ConcurrentHashMap + ConcurrentLinkedDeque) for multi-threaded environments, or keep it only single-threaded?**

```java
import java.util.*;

public class LRUCache<K, V> {
    private final int capacity;
    private final Map<K, V> map;
    private final Deque<K> deque; // to track usage order

    public LRUCache(int capacity) {
        this.capacity = capacity;
        this.map = new HashMap<>();
        this.deque = new ArrayDeque<>();
    }

    public V get(K key) {
        if (!map.containsKey(key)) {
            return null; // cache miss
        }
        // move accessed key to front (most recent)
        deque.remove(key);
        deque.addFirst(key);
        return map.get(key);
    }

    public void put(K key, V value) {
        if (map.containsKey(key)) {
            // update value and move to front
            map.put(key, value);
            deque.remove(key);
            deque.addFirst(key);
            return;
        }

        if (deque.size() == capacity) {
            // evict least recently used
            K lru = deque.removeLast();
            map.remove(lru);
        }

        // insert new
        deque.addFirst(key);
        map.put(key, value);
    }

    public void display() {
        for (K key : deque) {
            System.out.println(key + " => " + map.get(key));
        }
    }

    public static void main(String[] args) {
        LRUCache<Integer, String> cache = new LRUCache<>(3);

        cache.put(1, "one");
        cache.put(2, "two");
        cache.put(3, "three");
        cache.put(4, "four"); // evicts 1
        cache.get(3);         // moves 3 to front
        cache.get(2);         // moves 2 to front
        cache.put(1, "one");  // evicts 4
        cache.get(3);         // moves 3 to front
        cache.get(1);         // moves 1 to front

        cache.display();
        /*
         Output:
         1 => one
         3 => three
         2 => two
        */
    }
}

```

<img width="795" height="278" alt="image" src="https://github.com/user-attachments/assets/5c611a59-51a4-427c-8be7-b4cb4eed98b6" />
* For more details - > https://chatgpt.com/c/68d9d9f3-24fc-8322-9f45-ff7f098626b8

