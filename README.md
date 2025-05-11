# Linux-IPC-Semaphores
Ex05-Linux IPC-Semaphores

# AIM:
To Write a C program that implements a producer-consumer system with two processes using Semaphores.

# DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux Process API - Sempahores

### Step 3:

Execute the C Program for the desired output. 

# PROGRAM:

## Write a C program that implements a producer-consumer system with two processes using Semaphores.
```
/*
 * sem.c - Producer-Consumer using Semaphores (Persistent Semaphore Version)
 */

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>
#include <sys/wait.h>
#include <signal.h>
#include <fcntl.h>

#define NUM_LOOPS 10
#define SEM_KEY_PATH "./semkeyfile"
#define SEM_PROJ_ID 65

union semun {
    int val;
    struct semid_ds *buf;
    unsigned short int *array;
    struct seminfo *__buf;
};

int sem_set_id;

void wait_semaphore(int sem_set_id) {
    struct sembuf sem_op = {0, -1, 0};
    if (semop(sem_set_id, &sem_op, 1) == -1) {
        perror("semop - wait");
        exit(1);
    }
}

void signal_semaphore(int sem_set_id) {
    struct sembuf sem_op = {0, 1, 0};
    if (semop(sem_set_id, &sem_op, 1) == -1) {
        perror("semop - signal");
        exit(1);
    }
}

int main() {
    union semun sem_val;
    int child_pid;

    // Create a key using ftok()
    if (access(SEM_KEY_PATH, F_OK) == -1) {
        FILE *fp = fopen(SEM_KEY_PATH, "w");
        if (fp) fclose(fp);
    }
    key_t key = ftok(SEM_KEY_PATH, SEM_PROJ_ID);
    if (key == -1) {
        perror("ftok");
        exit(1);
    }

    // Try to get existing semaphore or create new one
    sem_set_id = semget(key, 1, IPC_CREAT | IPC_EXCL | 0600);
    if (sem_set_id == -1) {
        // If already exists, try accessing it
        sem_set_id = semget(key, 1, 0600);
        if (sem_set_id == -1) {
            perror("semget");
            exit(1);
        } else {
            printf("Attached to existing semaphore, id: %d\n", sem_set_id);
        }
    } else {
        // New semaphore, so initialize it
        printf("Created new semaphore, id: %d\n", sem_set_id);
        sem_val.val = 0;
        if (semctl(sem_set_id, 0, SETVAL, sem_val) == -1) {
            perror("semctl - SETVAL");
            exit(1);
        }
    }

    // Fork process
    child_pid = fork();
    if (child_pid < 0) {
        perror("fork");
        exit(1);
    }

    if (child_pid == 0) {
        // Child: Consumer
        for (int i = 0; i < NUM_LOOPS; i++) {
            wait_semaphore(sem_set_id);
            printf("consumer: '%d'\n", i);
            fflush(stdout);
        }
        exit(0);
    } else {
        // Parent: Producer
        for (int i = 0; i < NUM_LOOPS; i++) {
            printf("producer: '%d'\n", i);
            fflush(stdout);
            signal_semaphore(sem_set_id);
            usleep(500000);
        }

        wait(NULL);
        printf("Execution complete. Semaphore NOT removed.\n");
    }

    return 0;
}


```
## OUTPUT
$ ./sem.o 

![image](https://github.com/user-attachments/assets/8f86a52e-6da2-4832-a9db-1136e1e55cf6)


$ ipcs -s

![image](https://github.com/user-attachments/assets/38afd1ca-118c-4582-a1b3-ba2d3e66c646)


# RESULT:
The program is executed successfully.
