# Bank Teller Queue System

A console Java program that uses a **queue (FIFO)** to serve bank customers in the order they arrive.

## Why a queue?
Customers must be served fairly: first come, first served. A queue gives O(1) enqueue (join) and dequeue (serve), and buffers the gap between how fast customers arrive and how fast tellers work.

## Real-world applications of queues
- Bank and hospital service lines
- Printer spooling
- CPU process scheduling
- Call centers
- Web server request handling and message brokers
- Keyboard/mouse input buffers
- Breadth-first search
- Ticket booking and online checkout

## Files
- `BankQueueSystem.java` - the program (custom linked-list queue + menu)
- `index.html` - browser demo of the same queue (open in any browser)
- `README.md` - this file

## How to run the Java program
```
javac BankQueueSystem.java
java BankQueueSystem
```

## Features
1. Customer joins queue (gets a ticket number)
2. Call next customer (dequeue)
3. Show who is next (peek)
4. Display the waiting queue
5. Show statistics (served, waiting, average wait)
6. Exit

## Design
- `LinkedQueue<T>`: generic linked-list queue with `front` and `rear` pointers; methods `enqueue`, `dequeue`, `peek`, `isEmpty`, `size`, `display`.
- `Customer`: ticket number, name, service type, arrival time.

## Source code: BankQueueSystem.java

```java
import java.util.Scanner;

/**
 * Bank Teller Queue System
 * Demonstrates a FIFO queue (custom linked-list implementation)
 * used to serve customers in the order they arrive.
 */
public class BankQueueSystem {

    // ---------- Customer ticket ----------
    static class Customer {
        final int ticketNo;
        final String name;
        final String service;
        final long arrivalTime;

        Customer(int ticketNo, String name, String service) {
            this.ticketNo = ticketNo;
            this.name = name;
            this.service = service;
            this.arrivalTime = System.currentTimeMillis();
        }

        @Override
        public String toString() {
            return "#" + ticketNo + " " + name + " (" + service + ")";
        }
    }

    // ---------- Generic linked-list queue ----------
    static class LinkedQueue<T> {
        private class Node {
            T data;
            Node next;
            Node(T data) { this.data = data; }
        }

        private Node front, rear;
        private int size;

        // Add at the rear - O(1)
        void enqueue(T item) {
            Node node = new Node(item);
            if (rear == null) {
                front = rear = node;
            } else {
                rear.next = node;
                rear = node;
            }
            size++;
        }

        // Remove from the front - O(1)
        T dequeue() {
            if (isEmpty()) return null;
            T item = front.data;
            front = front.next;
            if (front == null) rear = null;
            size--;
            return item;
        }

        // Look at the front without removing - O(1)
        T peek() {
            return isEmpty() ? null : front.data;
        }

        boolean isEmpty() { return size == 0; }

        int size() { return size; }

        void display() {
            if (isEmpty()) {
                System.out.println("  (queue is empty)");
                return;
            }
            int position = 1;
            for (Node cur = front; cur != null; cur = cur.next) {
                System.out.println("  " + position++ + ". " + cur.data);
            }
        }
    }

    // ---------- Main program ----------
    private static final LinkedQueue<Customer> queue = new LinkedQueue<>();
    private static int nextTicket = 1;
    private static int served = 0;
    private static long totalWaitMs = 0;

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        boolean running = true;

        while (running) {
            System.out.println("\n===== BANK TELLER QUEUE SYSTEM =====");
            System.out.println("1. Customer joins queue");
            System.out.println("2. Call next customer");
            System.out.println("3. Show who is next");
            System.out.println("4. Display waiting queue");
            System.out.println("5. Show statistics");
            System.out.println("6. Exit");
            System.out.print("Choose: ");

            String choice = in.nextLine().trim();
            switch (choice) {
                case "1": joinQueue(in); break;
                case "2": callNext(); break;
                case "3": showNext(); break;
                case "4":
                    System.out.println("Waiting customers (" + queue.size() + "):");
                    queue.display();
                    break;
                case "5": showStats(); break;
                case "6": running = false; break;
                default: System.out.println("Invalid choice.");
            }
        }
        System.out.println("System closed. Goodbye.");
    }

    private static void joinQueue(Scanner in) {
        System.out.print("Customer name: ");
        String name = in.nextLine().trim();
        if (name.isEmpty()) {
            System.out.println("Name cannot be empty.");
            return;
        }
        System.out.println("Service: 1) Deposit  2) Withdrawal  3) Account Enquiry  4) Loan");
        System.out.print("Choose service: ");
        String service;
        switch (in.nextLine().trim()) {
            case "1": service = "Deposit"; break;
            case "2": service = "Withdrawal"; break;
            case "3": service = "Account Enquiry"; break;
            case "4": service = "Loan"; break;
            default:
                System.out.println("Invalid service.");
                return;
        }
        Customer c = new Customer(nextTicket++, name, service);
        queue.enqueue(c);
        System.out.println("Ticket issued: " + c + ". People ahead: " + (queue.size() - 1));
    }

    private static void callNext() {
        Customer c = queue.dequeue();
        if (c == null) {
            System.out.println("No customers waiting.");
            return;
        }
        long waited = System.currentTimeMillis() - c.arrivalTime;
        totalWaitMs += waited;
        served++;
        System.out.println("Now serving: " + c + " | waited " + (waited / 1000) + "s");
    }

    private static void showNext() {
        Customer c = queue.peek();
        System.out.println(c == null ? "No customers waiting." : "Next in line: " + c);
    }

    private static void showStats() {
        System.out.println("Customers served : " + served);
        System.out.println("Still waiting    : " + queue.size());
        if (served > 0) {
            System.out.println("Average wait     : " + (totalWaitMs / served / 1000) + "s");
        }
    }
}
```