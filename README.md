---
author: Mathias Østby
email: mathiasoestby@gmail.com
---
> Author: Mathias Østby (maostby)

# Obligatory task 1

## Task 1

### Find-routine

$$V_1 = \{\text{head, head.val}, \text{head.next}\}$$
$$W_1 = Ø$$

### Insert-routine

$$V_2 = \{\text{head, tail, tail.next}\}$$
$$W_2 = \{\text{head, tail, tail.next}\}$$

### delfront-routine

$$V_3 = \{\text{head}, \text{head.next}, \text{tail}\}$$
$$W_3 = \{\text{head, tail}\}$$


## Task 2

### a)
We look at the intersections between global variables read and written between the routines: 

$$
\begin{align*} 
    V_1 \cup W_2 &= \{\text{head}\}\\ 
    V_1 \cup W_3 &=\{\text{head}\}\\\\
    V_2 \cup W_1 &= Ø\\
    V_2 \cup W_3 &=\{\text{head, tail}\}\\\\
    V_3 \cup W_1 &= Ø\\
    V_3 \cup W_2 &=\{\text{head, tail}\}
\end{align*}
$$

We can see that no to routines achieve the interference criterion $V_A\cap W_B = V_B\cap W_A = Ø $.

> im not quite sure what i have gotten wrong here, but I see that something is wrong

## Task 3
See supplied code in "task3/no/uio/ifi/task"

Main.java
```java
package no.uio.ifi.task;

import java.util.HashMap;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;


/* You are allowed to 1. add modifiers to fields and method signatures of subclasses, 
and 2. add code at the marked places, including removing the following return */
public class Main {
    public static void main(String[] args) throws InterruptedException {

        LinkedQueue<Integer> inputQueue = new LinkedQueue<>();
        LinkedQueue<Integer> evenQueue = new LinkedQueue<>();
        LinkedQueue<Integer> oddQueue = new LinkedQueue<>();

        HashMap<Boolean, LinkedQueue<Integer>> layer = new HashMap<>();
        layer.put(true, evenQueue);
        layer.put(false, oddQueue);

        int n = 1000;

        ExecutorService inputExc = Executors.newCachedThreadPool();
        // start n threads, each adding a single number to inputQueue
        for (int i = 0; i < n; i++) {
            final int finalI = i;
            inputExc.execute(() -> inputQueue.insert(finalI));
        }


        Mapper<Integer, Boolean> mapper1 = new Mapper<Integer, Boolean>(layer) {
            @Override
            void transform(Integer input) {
                if (input == null) {
                    return;
                }
                //take number and put it into the right queue
                input = input * input;
                if (input % 2 == 0) {
                    evenQueue.insert(input);
                } else {
                    oddQueue.insert(input);
                }
            }
        };
        Mapper<Integer, Boolean> mapper2 = new Mapper<Integer, Boolean>(layer) {
            @Override
            void transform(Integer input) {
                if (input == null) {
                    return;
                }
                // take number and put it into the right queue 
                input = input * input;
                if (input % 2 == 0) {
                    evenQueue.insert(input);
                } else {
                    oddQueue.insert(input);
                }
            }
        };



        ExecutorService distribute = Executors.newCachedThreadPool();
        // start n threads, each taking a single number from inputQueue to either mapper1 or mapper2
        // *        each mapper must have the same amount of work
        // *        the mapper must add its number to the correct queue*/
        for (int i = 0; i < n; i++) {
            if (i % 2 == 0) {
                distribute.execute(() -> mapper1.transform(inputQueue.delfront()));
            } else {
                distribute.execute(() -> mapper2.transform(inputQueue.delfront()));
            }
        }

        Reducer<Integer> reducer1 = new Reducer<Integer>() {
            @Override
            protected void reduce(Integer input) {
                // implement me */
                if (input == null) {
                    return;
                }
                synchronized (this) {
                    current += input;
                }
            }
        };
        Reducer<Integer> reducer2 = new Reducer<Integer>() {

            @Override
            protected void reduce(Integer input) {
                /* implement me */
                if (input == null) {
                    return;
                }
                synchronized (this) {
                    current += input;
                }

            }
        };


        ExecutorService reduce = Executors.newCachedThreadPool();

        //start n threads, each taking one number from either queue and giving it to a reducer.
        //        Reducer 1 will only add even numbers, reducer 2 will only add odd numbers */
        for (int i = 0; i < n; i++) {
            if (i % 2 == 0) {
                reduce.execute(() -> reducer1.reduce(evenQueue.delfront()));                    
            } else {
                reduce.execute(() -> reducer2.reduce(oddQueue.delfront()));
            }
        }

        Thread.sleep(2000);
        System.out.println("Sum even: "+reducer1.current);
        System.out.println("Sum odd: "+reducer2.current);

    }
}
```

LinkedQueue.java
```java
package no.uio.ifi.task;
/* You are allowed to 1. add modifiers to fields and method signatures and
 2. add code at the marked places, including removing the following return */


public class LinkedQueue<T> {
    private Node<T> head = null;
    private Node<T> tail = null;


    public synchronized int find(T t){
        // implement me
        Node<T> i = head;

        while (i != null && i.content != t) {
            i = i.next;
        }
        return i == null ? -1 : 0;
    }

    public void insert(T t){
        /* implement me */

        Node<T> newNode = new Node<T>(t);
        
        if (tail == null) {
            head = newNode;
        } else {
            tail.next = newNode;
        }
        tail = newNode;
    }

    public synchronized T delfront(){
        // implement me */

        if (head != null) {
            if (head == tail) {
                tail = null;
            }
            Node<T> oldHead = head;
            head = head.next;

            return oldHead.content;
        }
        return null;
    }
}

```
