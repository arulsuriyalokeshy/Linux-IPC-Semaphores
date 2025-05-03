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
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <time.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>

#define NUM_LOOPS 20

#if defined(__GNU_LIBRARY__) && !defined(_SEM_SEMUN_UNDEFINED)
#else
union semun {
    int val;
    struct semid_ds *buf;
    unsigned short int *array;
    struct seminfo *__buf;
};
#endif

int main(int argc, char *argv[]) {
    int sem_set_id;
    union semun sem_val;
    int child_pid;
    int i;
    struct sembuf sem_op;
    int rc;
    struct timespec delay;

    sem_set_id = semget(IPC_PRIVATE, 1, 0600);
    if (sem_set_id == -1) {
        perror("semget");
        exit(1);
    }

    printf("semaphore set created, semaphore set id '%d'.\n", sem_set_id);

    sem_val.val = 0;
    rc = semctl(sem_set_id, 0, SETVAL, sem_val);
    if (rc == -1) {
        perror("semctl");
        exit(1);
    }

    child_pid = fork();

    if (child_pid == -1) {
        perror("fork");
        exit(1);
    } else if (child_pid == 0) {
        for (i = 0; i < NUM_LOOPS; i++) {
            sem_op.sem_num = 0;
            sem_op.sem_op = -1;
            sem_op.sem_flg = 0;
            if (semop(sem_set_id, &sem_op, 1) == -1) {
                perror("semop - child");
                exit(1);
            }
            printf("consumer: '%d'\n", i);
            fflush(stdout);
        }
    } else {
        srand(time(NULL));
        for (i = 0; i < NUM_LOOPS; i++) {
            printf("producer: '%d'\n", i);
            fflush(stdout);
            sem_op.sem_num = 0;
            sem_op.sem_op = 1;
            sem_op.sem_flg = 0;
            if (semop(sem_set_id, &sem_op, 1) == -1) {
                perror("semop - parent");
                exit(1);
            }
            if (rand() > 3 * (RAND_MAX / 4)) {
                delay.tv_sec = 0;
                delay.tv_nsec = 10000000;
                nanosleep(&delay, NULL);
            }
        }
        semctl(sem_set_id, 0, IPC_RMID, sem_val);
    }

    return 0;
}

```





## OUTPUT


![image](https://github.com/user-attachments/assets/9826facd-b6e0-4607-afee-30a1a221866f)

![image](https://github.com/user-attachments/assets/4f876c0a-a409-4e42-8bff-2c3a3359725c)



# RESULT:
The program is executed successfully.
