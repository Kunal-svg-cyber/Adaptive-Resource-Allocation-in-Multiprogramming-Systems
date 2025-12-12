#include <bits/stdc++.h>
using namespace std;

// --------------------- PROCESS STRUCTURE ---------------------
struct Process {
    int pid;
    int burstTime;
    int remainingTime;
    int priority;
    int memoryRequired;
    int waitingTime = 0;
    int turnaroundTime = 0;
    string state; // Ready, Running, Waiting, Finished
};

// --------------------- SYSTEM METRICS ---------------------
struct Metrics {
    int totalCPUTime = 0;
    int idleTime = 0;
    int totalMemory = 1024; // assume 1 GB memory
    int usedMemory = 0;
    float cpuUtilization = 0;
    float memoryUtilization = 0;
};

// --------------------- ADAPTIVE SCHEDULER ---------------------
class AdaptiveScheduler {
private:
    vector<Process> processes;
    queue<int> readyQueue;
    Metrics metrics;
    int timeQuantum;
    int currentTime;

public:
    AdaptiveScheduler(int tq) {
        timeQuantum = tq;
        currentTime = 0;
    }

    void addProcess(Process p) {
        processes.push_back(p);
        readyQueue.push(p.pid);
        metrics.usedMemory += p.memoryRequired;
    }

    void displayProcesses() {
        cout << "\nPID\tBurst\tRemain\tPriority\tMem\tState\n";
        for (auto &p : processes) {
            cout << p.pid << "\t" << p.burstTime << "\t" << p.remainingTime
                 << "\t" << p.priority << "\t\t" << p.memoryRequired
                 << "\t" << p.state << endl;
        }
    }

    void simulate() {
        cout << "\n=== Starting Simulation ===\n";

        while (!readyQueue.empty()) {
            int pid = readyQueue.front();
            readyQueue.pop();

            Process &p = processes[pid];
            p.state = "Running";
            cout << "\n[Time " << currentTime << "] Running Process " << p.pid << endl;

            int execTime = min(timeQuantum, p.remainingTime);
            currentTime += execTime;
            metrics.totalCPUTime += execTime;
            p.remainingTime -= execTime;

            // Update waiting times
            for (auto &q : processes) {
                if (q.state == "Ready") q.waitingTime += execTime;
            }

            // Adaptive logic: modify timeQuantum and priorities dynamically
            adaptResources();

            if (p.remainingTime == 0) {
                p.state = "Finished";
                p.turnaroundTime = currentTime;
                cout << "Process " << p.pid << " finished at time " << currentTime << endl;
            } else {
                p.state = "Ready";
                readyQueue.push(p.pid);
            }

            updateMetrics();
            displayProcesses();
        }

        showSummary();
    }

    void adaptResources() {
        // Adaptive logic: adjust time quantum or priorities
        int totalWaiting = 0, finished = 0;
        for (auto &p : processes) {
            if (p.state != "Finished") totalWaiting += p.waitingTime;
            else finished++;
        }
        int activeCount = processes.size() - finished;
        int avgWaiting = (activeCount > 0) ? totalWaiting / activeCount : 0;

        // Adjust time quantum dynamically
        if (avgWaiting > 10) {
            timeQuantum = max(2, timeQuantum - 1);
        } else if (avgWaiting < 5) {
            timeQuantum = min(8, timeQuantum + 1);
        }

        // Adjust priorities if someone is starving
        for (auto &p : processes) {
            if (p.waitingTime > 15 && p.state == "Ready")
                p.priority++;
        }

        cout << "[Adaptive] New Time Quantum: " << timeQuantum << endl;
    }

    void updateMetrics() {
        metrics.cpuUtilization = (float)metrics.totalCPUTime /
                                 (float)(currentTime) * 100;
        metrics.memoryUtilization = (float)metrics.usedMemory /
                                    (float)metrics.totalMemory * 100;
    }

    void showSummary() {
        cout << "\n=== Simulation Summary ===\n";
        for (auto &p : processes) {
            cout << "Process " << p.pid
                 << " -> Waiting: " << p.waitingTime
                 << ", Turnaround: " << p.turnaroundTime << endl;
        }
        cout << "\nCPU Utilization: " << metrics.cpuUtilization << "%";
        cout << "\nMemory Utilization: " << metrics.memoryUtilization << "%";
        cout << "\nSimulation Ended at time: " << currentTime << endl;
    }
};

// --------------------- MAIN FUNCTION ---------------------
int main() {
    AdaptiveScheduler scheduler(4);

    // Sample processes
    Process p1 = {0, 12, 12, 1, 200, 0, 0, "Ready"};
    Process p2 = {1, 8, 8, 2, 300, 0, 0, "Ready"};
    Process p3 = {2, 15, 15, 1, 250, 0, 0, "Ready"};

    scheduler.addProcess(p1);
    scheduler.addProcess(p2);
    scheduler.addProcess(p3);

    scheduler.displayProcesses();
    scheduler.simulate();

    return 0;
}


