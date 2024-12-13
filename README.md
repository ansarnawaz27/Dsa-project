#include <iostream>
#include <string>
#include <fstream>
#include <sstream>
using namespace std;

class Event {
public:
    string name;
    int time;
    int priority;

    Event(string name, int time, int priority) : name(name), time(time), priority(priority) {}
    Event() : name(""), time(0), priority(0) {}

    string toFileString() const {
        stringstream ss;
        ss << name << "," << time << "," << priority;
        return ss.str();
    }

    static Event fromFileString(const string& str) {
        stringstream ss(str);
        string name;
        int time, priority;
        getline(ss, name, ',');
        ss >> time;
        ss.ignore();
        ss >> priority;
        return Event(name, time, priority);
    }
};

struct Node {
    Event event;
    Node* next;
    Node(Event event) : event(event), next(nullptr) {}
};

class PriorityQueue {
private:
    Node* head;

public:
    PriorityQueue() : head(nullptr) {}

    void insert(Event event) {
        Node* newNode = new Node(event);
        if (!head || head->event.priority > event.priority) {
            newNode->next = head;
            head = newNode;
        }
        else {
            Node* temp = head;
            while (temp->next && temp->next->event.priority <= event.priority) {
                temp = temp->next;
            }
            newNode->next = temp->next;
            temp->next = newNode;
        }
    }

    Event remove() {
        if (!head) throw runtime_error("Queue is empty.");
        Node* temp = head;
        head = head->next;
        Event event = temp->event;
        delete temp;
        return event;
    }

    Event peek() {
        if (!head) throw runtime_error("Queue is empty.");
        return head->event;
    }

    void display() {
        Node* temp = head;
        if (!temp) {
            cout << "Priority Queue is empty.\n";
            return;
        }
        while (temp) {
            cout << "Event: " << temp->event.name
                << ", Time: " << temp->event.time
                << ", Priority: " << temp->event.priority << endl;
            temp = temp->next;
        }
    }

    bool isEmpty() {
        return head == nullptr;
    }

    int count() {
        int counter = 0;
        Node* temp = head;
        while (temp) {
            counter++;
            temp = temp->next;
        }
        return counter;
    }

    void clear() {
        while (head) {
            Node* temp = head;
            head = head->next;
            delete temp;
        }
        cout << "Priority Queue cleared.\n";
    }

    void saveToFile(const string& filename) {
        ofstream file(filename);
        if (file.is_open()) {
            Node* temp = head;
            while (temp) {
                file << temp->event.toFileString() << endl;
                temp = temp->next;
            }
            file.close();
            cout << "Events saved to file: " << filename << endl;
        }
        else {
            cout << "Unable to open file for saving.\n";
        }
    }

    void loadFromFile(const string& filename) {
        ifstream file(filename);
        if (file.is_open()) {
            string line;
            while (getline(file, line)) {
                Event event = Event::fromFileString(line);
                insert(event);
            }
            file.close();
            cout << "Events loaded from file: " << filename << endl;
        }
        else {
            cout << "Unable to open file for loading.\n";
        }
    }
};

int main() {
    PriorityQueue pq;
    pq.insert(Event("Meeting with Client", 10, 2));
    pq.insert(Event("Lunch Break", 12, 1));
    pq.insert(Event("Project Deadline", 15, 1));
    pq.insert(Event("Team Discussion", 14, 3));

    cout << "Events in Priority Queue:\n";
    pq.display();
    cout << endl;

    pq.saveToFile("events.txt");

    cout << "Peeking at the highest-priority event:\n";
    Event peekedEvent = pq.peek();
    cout << "Event: " << peekedEvent.name
        << ", Time: " << peekedEvent.time
        << ", Priority: " << peekedEvent.priority << endl << endl;

    cout << "Total events in the queue: " << pq.count() << endl << endl;

    Event removedEvent = pq.remove();
    cout << "Removed Event: " << removedEvent.name
        << ", Time: " << removedEvent.time
        << ", Priority: " << removedEvent.priority << endl << endl;

    cout << "Remaining Events in Priority Queue:\n";
    pq.display();
    cout << endl;

    pq.clear();

    pq.loadFromFile("events.txt");

    cout << "Is the queue empty? " << (pq.isEmpty() ? "Yes" : "No") << endl;

    return 0;
}
