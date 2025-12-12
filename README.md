
// Operating Systems Project, ( 3737 )
// Hebah Fwaz althbyani ********
// Renad saad Alosiami ********
// Manar Nasser Alosaimi ********
// Shahad Mansour Alshehri ********
// teaf faisal alharthy   ********
// Dr. Nada Al-Tuwaigri
// =======================================

#include <iostream>
#include <vector>
#include <queue>
#include <map>
#include <iomanip>
#include <algorithm>
#include <climits>
using namespace std;

// =======================================
//             Part 1: CPU Scheduling
// =======================================

struct PCB { int pid, at, bt, pr; };
struct Block { int from, to, pid; };

void showGantt(const vector<Block>& timeline) {
    cout << "\n--- Gantt Chart ---\n";
    for (auto &b : timeline) cout << "| P" << b.pid << " ";
    cout << "|\n";
    if (!timeline.empty()) {
        cout << timeline.front().from << " ";
        for (auto &b : timeline) cout << b.to << " ";
    }
    cout << "\n";
}

void showTable(const string& title, const vector<PCB>& arr,
               const map<int,int>& finish,
               const map<int,int>& first) {

    cout << "\n=== " << title << " ===\n";
    cout << left << setw(5) << "PID" << setw(6) << "AT" 
         << setw(6) << "BT" << setw(6) << "PR" << setw(6)
         << "CT" << setw(6) << "TAT" << setw(6)
         << "WT" << setw(6) << "RT" << endl;

    double sumWT=0, sumTAT=0, sumRT=0;
    int n = arr.size();

    for (auto &p : arr) {
        int ct = finish.at(p.pid);
        int tat = ct - p.at;
        int wt = tat - p.bt;
        int rt = first.at(p.pid) - p.at;

        sumWT += wt;
        sumTAT += tat;
        sumRT += rt;

        cout << "P" << p.pid << "   "
             << setw(6) << p.at << setw(6)
             << p.bt << setw(6) << p.pr << setw(6)
             << ct << setw(6) << tat << setw(6)
             << wt << setw(6) << rt << endl;
    }

    cout << "\nAverage WT = " << sumWT/n << endl;
    cout << "Average TAT = " << sumTAT/n << endl;
    cout << "Average RT = " << sumRT/n << endl;
}

void FCFS(const vector<PCB>& input) {
    vector<PCB> p = input;
    sort(p.begin(), p.end(), [](auto&a, auto&b){ return a.at < b.at; });

    int t = 0;
    map<int,int> finish, first;
    vector<Block> blocks;

    for (auto &pr : p) {
        if (t < pr.at) t = pr.at;
        first[pr.pid] = t;
        int start = t;
        t += pr.bt;
        finish[pr.pid] = t;
        blocks.push_back({start, t, pr.pid});
    }

    showGantt(blocks);
    showTable("FCFS", input, finish, first);
}

void SJF(const vector<PCB>& input) {
    vector<PCB> p = input;
    int n = p.size(), t = 0, done = 0;
    map<int,int> finish, first;
    vector<bool> used(n, false);
    vector<Block> blocks;

    while (done < n) {
        int idx = -1;
        for (int i=0;i<n;i++)
            if (!used[i] && p[i].at <= t)
                if (idx == -1 || p[i].bt < p[idx].bt) idx = i;

        if (idx == -1) {
            int next = INT_MAX;
            for (int i=0;i<n;i++) if (!used[i]) next = min(next, p[i].at);
            t = next;
            continue;
        }

        first[p[idx].pid] = t;
        int start = t;
        t += p[idx].bt;
        finish[p[idx].pid] = t;
        blocks.push_back({start,t,p[idx].pid});
        used[idx] = true;
        done++;
    }

    showGantt(blocks);
    showTable("SJF", input, finish, first);
}

void PriorityNonPre(const vector<PCB>& input) {
    vector<PCB> p = input;
    int n = p.size(), t = 0, done = 0;
    map<int,int> finish, first;
    vector<bool> used(n, false);
    vector<Block> blocks;

    while (done < n) {
        int idx = -1;
        for (int i=0;i<n;i++)
            if (!used[i] && p[i].at <= t)
                if (idx == -1 || p[i].pr < p[idx].pr) idx = i;

        if (idx == -1) {
            int next = INT_MAX;
            for (int i=0;i<n;i++) if (!used[i]) next = min(next, p[i].at);
            t = next;
            continue;
        }

        first[p[idx].pid] = t;
        int start = t;
        t += p[idx].bt;
        finish[p[idx].pid] = t;
        blocks.push_back({start,t,p[idx].pid});
        used[idx] = true;
        done++;
    }

    showGantt(blocks);
    showTable("Priority", input, finish, first);
}

void RR(const vector<PCB>& input, int q) {
    vector<PCB> p = input;
    sort(p.begin(), p.end(), [](auto&a, auto&b){ return a.at < b.at; });
    map<int,int> rem, finish, first;
    for (auto &x : p) rem[x.pid] = x.bt;

    queue<int> Q;
    vector<Block> blocks;

    int t = 0, idx = 0, n = p.size();

    while (finish.size() < (size_t)n) {
        while (idx < n && p[idx].at <= t) {
            Q.push(p[idx].pid);
            idx++;
        }

        if (Q.empty()) {
            t = p[idx].at;
            continue;
        }

        int pid = Q.front();
        Q.pop();

        PCB cur;
        for (auto &x : p) if (x.pid == pid) cur = x;

        if (!first.count(pid)) first[pid] = t;

        int run = min(q, rem[pid]);
        int start = t;
        t += run;
        int end = t;

        blocks.push_back({start,end,pid});
        rem[pid] -= run;

        while (idx < n && p[idx].at <= t) {
            Q.push(p[idx].pid);
            idx++;
        }

        if (rem[pid] == 0) finish[pid] = t;
        else Q.push(pid);
    }

    showGantt(blocks);
    showTable("Round Robin", input, finish, first);
}

// =======================================
//         Part 2: Banker’s Algorithm
// =======================================

void bankers() {
    int n, m;
    cout << "Enter number of processes: ";
    cin >> n;
    cout << "Enter number of resource types: ";
    cin >> m;

    vector<vector<int>> maxD(n, vector<int>(m));
    vector<vector<int>> alloc(n, vector<int>(m));
    vector<int> avail(m);

    cout << "\nEnter MAX matrix (each row in one line):\n";
    for(int i = 0; i < n; i++){
        cout << "Max for P" << i << ": ";
        for(int j = 0; j < m; j++) cin >> maxD[i][j];
    }

    cout << "\nEnter ALLOCATION matrix:\n";
    for(int i = 0; i < n; i++){
        cout << "Allocation for P" << i << ": ";
        for(int j = 0; j < m; j++) cin >> alloc[i][j];
    }

    cout << "\nEnter AVAILABLE vector:\nAvailable: ";
    for(int j = 0; j < m; j++) cin >> avail[j];

    vector<vector<int>> need(n, vector<int>(m));
    for(int i=0;i<n;i++)
        for(int j=0;j<m;j++)
            need[i][j] = maxD[i][j] - alloc[i][j];

    vector<int> work = avail;
    vector<bool> finished(n, false);
    vector<int> safeSeq;

    cout << "\nNeed Matrix:\n";
    int count = 0;
    while(count < n){
        bool found = false;
        for(int i=0;i<n;i++){
            if(!finished[i]){
                bool canExec = true;
                for(int j=0;j<m;j++)
                    if(need[i][j] > work[j]){
                        canExec = false;
                        break;
                    }
                if(canExec){
                    cout << "Process P" << i << " can be executed (Need <= Work).\n";
                    cout << "Work before: ";
                    for(int k=0;k<m;k++) cout << work[k] << " ";
                    cout << "\n";

                    for(int j=0;j<m;j++) work[j] += alloc[i][j];

                    cout << "Work after: ";
                    for(int k=0;k<m;k++) cout << work[k] << " ";
                    cout << "\n\n";

                    safeSeq.push_back(i);
                    finished[i] = true;
                    found = true;
                    count++;
                }
            }
        }
        if(!found) break;
    }

    if(count == n){
        cout << "System is in a SAFE state.\nSafe sequence: ";
        for(int i=0;i<safeSeq.size();i++){
            cout << "P" << safeSeq[i];
            if(i != safeSeq.size()-1) cout << " -> ";
        }
        cout << endl;
    } else {
        cout << "System is in an UNSAFE state.\n";
    }
}

// =======================================
//                 Main
// =======================================

int main() {
    cout << "=== Operating Systems Project ===\n";
    cout << "Choose option:\n1. CPU Scheduling\n2. Banker’s Algorithm\n3. Both\nYour choice: ";
    int choice;
    cin >> choice;

    if(choice == 1 || choice == 3){
        int n;
        cout << "\nEnter number of processes for CPU Scheduling: ";
        cin >> n;
        vector<PCB> processes(n);
        for(int i=0;i<n;i++){
            processes[i].pid = i;
            cout << "\nProcess P" << i << "\nArrival Time: ";
            cin >> processes[i].at;
            cout << "Burst Time: ";
            cin >> processes[i].bt;
            cout << "Priority: ";
            cin >> processes[i].pr;
        }
        int q;
        cout << "\nEnter Time Quantum for Round Robin: ";
        cin >> q;

        FCFS(processes);
        SJF(processes);
        PriorityNonPre(processes);
        RR(processes, q);
    }

    if(choice == 2 || choice == 3){
        bankers();
    }

    return 0;
}
