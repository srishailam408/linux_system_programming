/* Write a C program to create a child process using fork() and demonstrate process communication using shared memory and semaphores */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/sem.h>
#include <sys/wait.h>

#define SHM_SIZE 1024
union semun 
{
    int val;
    struct semid_ds *buf;
    unsigned short *array;
};

void sem_wait(int semid) 
{
    struct sembuf sb = {0, -1, 0}; 
    if (semop(semid, &sb, 1) == -1) 
    {
        perror("semop - wait");
        exit(1);
    }
}

void sem_signal(int semid) 
{
    struct sembuf sb = {0, 1, 0};
    if (semop(semid, &sb, 1) == -1) 
    {
        perror("semop - signal");
        exit(1);
    }
}

int main() 
{
    key_t key = ftok("file.txt", 65);
    if (key == -1) 
    {
        perror("ftok");
        exit(1);
    }
    int shmid = shmget(key, SHM_SIZE, 0666 | IPC_CREAT);
    if (shmid == -1) 
    {
        perror("shmget");
        exit(1);
    }
    int semid = semget(key, 1, 0666 | IPC_CREAT);
    if (semid == -1) 
    {
        perror("semget");
        shmctl(shmid, IPC_RMID, NULL);
        exit(1);
    }

    union semun sem_union;
    sem_union.val = 0;
    if (semctl(semid, 0, SETVAL, sem_union) == -1) 
    {
        perror("semctl SETVAL");
        shmctl(shmid, IPC_RMID, NULL);
        semctl(semid, 0, IPC_RMID);
        exit(1);
    }
    char *shared_mem = (char *)shmat(shmid, NULL, 0);
    if (shared_mem == (char *) -1) 
    {
        perror("shmat");
        shmctl(shmid, IPC_RMID, NULL);
        semctl(semid, 0, IPC_RMID);
        exit(1);
    }
    pid_t pid = fork();

    if (pid < 0) 
    {
        perror("fork");
        shmdt(shared_mem);
        shmctl(shmid, IPC_RMID, NULL);
        semctl(semid, 0, IPC_RMID);
        exit(1);
    }

    if (pid == 0) 
    {
        sem_wait(semid);
        printf("Child: Received message: %s\n", shared_mem);
        snprintf(shared_mem, SHM_SIZE, "Hello from child!");
        sem_signal(semid);
        shmdt(shared_mem);
        exit(0);
    } 
    else 
    {
        snprintf(shared_mem, SHM_SIZE, "Hello from parent!");
        sem_signal(semid);
        sem_wait(semid);
        printf("Parent: Received response: %s\n", shared_mem);
        wait(NULL);
        shmdt(shared_mem);
        shmctl(shmid, IPC_RMID, NULL);
        semctl(semid, 0, IPC_RMID);
    }

    return 0;
}

/* Write a program in C to create a zombie process and explain how to avoid it */

#include<stdio.h>
#include<unistd.h>
#include<sys/types.h>
#include<stdlib.h>
int main()
{
        int pid;
	pid=fork();
	if(pid<0)
	{
		perror("fork failed");
		return 1;
	}
	else if(pid==0)
	{
		printf("child process(PID: %d)is terminating\n",getpid());
		exit(0);
	}
	else
	{
		printf("parent process (PID: %d) is sleeping.\n",getpid());
		printf("check for zombie process: ps -aux | grep z\n");
		sleep(20);
		printf("parent work up.\n");
	}
	return 0;
}
/* Write a C program to demonstrate the use of the system() function for executing shell commands. */

#include<stdio.h>
#include<stdlib.h>
int main()
{
	int ret;
	printf("listing files in current directory using 'ls -l':\n\n");
	ret=system("ls -l");
	if(ret==-1)
	{
		perror("system() failed");
		return 1;
	}
	printf("\ndone executing shell command.\n");
	return 0;
}
/* Write a C program to demonstrate the use of the execvpe() function. */

#define _GNU_SOURCE 
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() 
{
    char *program = "/bin/ls";
    char *args[] = {"ls", "-l", "/tmp", NULL};
    char *envp[] = {"MYVAR=HelloWorld","PATH=/bin:/usr/bin",NULL};

    printf("Calling execvpe to execute %s...\n\n", program);

    if (execvpe(program, args, envp) == -1) 
    {
        perror("execvpe failed");
        return 1;
    }
    return 0;
}
/* Write a program in C to demonstrate inter-process communication (IPC) using shared memory. */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h> 
#include <sys/mman.h>
#include <sys/stat.h>
#include <unistd.h>
#include <sys/wait.h>

#define SHM_NAME "/my_shm"
#define SHM_SIZE 1024

int main() 
{
    int shm_fd;
    char *shm_ptr;
    shm_fd = shm_open(SHM_NAME, O_CREAT | O_RDWR, 0666);
    if (shm_fd == -1) 
    {
        perror("shm_open failed");
        exit(EXIT_FAILURE);
    }

    if (ftruncate(shm_fd, SHM_SIZE) == -1) 
    {
        perror("ftruncate failed");
        exit(EXIT_FAILURE);
    }
    shm_ptr = mmap(0, SHM_SIZE, PROT_READ | PROT_WRITE, MAP_SHARED, shm_fd, 0);
    if (shm_ptr == MAP_FAILED) 
    {
        perror("mmap failed");
        exit(EXIT_FAILURE);
    }

    pid_t pid = fork();

    if (pid < 0) 
    {
        perror("fork failed");
        exit(EXIT_FAILURE);
    } 
    else if (pid == 0) 
    {
        sleep(1);
        printf("Child read from shared memory: %s\n", shm_ptr);
        munmap(shm_ptr, SHM_SIZE);
        close(shm_fd);
        exit(0);
    } 
    else 
    {
        const char *message = "Hello from parent!";
        snprintf(shm_ptr, SHM_SIZE, "%s", message);
        printf("Parent wrote to shared memory: %s\n", message);

        wait(NULL);
        munmap(shm_ptr, SHM_SIZE);
        close(shm_fd);
        shm_unlink(SHM_NAME);
        printf("Parent cleaned up shared memory.\n");
    }

    return 0;
}
/* Write a program in C to demonstrate the use of the nice() system call for adjusting process priority. */

#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<sys/time.h>
#include<sys/resource.h>
#include<errno.h>
int main()
{
	int priority;
	errno=0;
	priority=getpriority(PRIO_PROCESS,0);
	if(priority==-1&&errno!=0)
	{
		perror("getpriority");
		return 1;
	}
	printf("current priority: %d\n",priority);
	int increment=5;
	errno=0;
	int new_priority=nice(increment);
	if(new_priority==-1&&errno!=0)
	{
		perror("nise");
		return 1;
	}
	printf("priority after nice(%d):%d\n",increment,new_priority);
	errno=0;
	priority=getpriority(PRIO_PROCESS,0);
	if(priority==-1&&errno!=0)
	{
		perror("getpriority");
		return 0;
	}
	printf("confirmed new priority: %d\n",priority);
	return 0;
}
/* Write a C program to demonstrate process synchronization using the fork() and wait() system calls. */

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int main() 
{
    pid_t pid;
    pid = fork();
    if (pid < 0) 
    {
        perror("Fork failed");
        return 1;
    }
    if (pid == 0) 
    {
        printf("Child process started. PID = %d\n", getpid());
        sleep(2);
        printf("Child process finished.\n");
        exit(0); 
    } 
    else 
    {
        printf("Parent process waiting for child to complete...\n");
        wait(NULL);
        printf("Parent process resumed after child completed.\n");
    }

    return 0;
}

/* Write a C program to create a child process using fork() and demonstrate inter-process communication (IPC) using shared memory. */

#include <stdio.h>
#include <stdlib.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/types.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>

#define SHM_SIZE 1024 

int main() 
{
    key_t key;
    int shmid;
    char *shared_memory;
    key = ftok("file.txt", 65); 
    if (key == -1) 
    {
        perror("ftok");
        exit(1);
    }

    shmid = shmget(key, SHM_SIZE, 0666 | IPC_CREAT);
    if (shmid < 0) 
    {
        perror("shmget");
        exit(1);
    }

    pid_t pid = fork();

    if (pid < 0) 
    {
        perror("fork");
        exit(1);
    }

    if (pid == 0) 
    {
        shared_memory = (char *)shmat(shmid, NULL, 0);
        if (shared_memory == (char *)(-1)) 
	{
            perror("shmat (child)");
            exit(1);
        }

        printf("Child: Reading from shared memory...\n");
        printf("Child read: %s\n", shared_memory);
        strcpy(shared_memory, "Hello from child!");
        shmdt(shared_memory);
    } 
    else 
    {
        shared_memory = (char *)shmat(shmid, NULL, 0);
        if (shared_memory == (char *)(-1)) 
	{
            perror("shmat (parent)");
            exit(1);
        }

        strcpy(shared_memory, "Hello from parent!");
        printf("Parent wrote to shared memory.\n");
        wait(NULL);
        printf("Parent: Reading updated memory: %s\n", shared_memory);
        shmdt(shared_memory);
        shmctl(shmid, IPC_RMID, NULL);
    }

    return 0;
}
/* Write a C program to create a child process using fork() and demonstrate process communication using message queues. */

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/ipc.h>
#include <sys/msg.h>
#include <string.h>
#include <sys/wait.h>

struct msg_buffer 
{
    long msg_type;
    char msg_text[100];
};

int main() 
{
    key_t key;
    int msgid;
    struct msg_buffer message;
    key = ftok("file.txt", 65); 
    if (key == -1) 
    {
        perror("ftok");
        exit(1);
    }
    msgid = msgget(key, 0666 | IPC_CREAT);
    if (msgid < 0) 
    {
        perror("msgget");
        exit(1);
    }
    pid_t pid = fork();
    if (pid < 0) 
    {
        perror("fork failed");
        exit(1);
    }
    if (pid == 0) 
    {
        msgrcv(msgid, &message, sizeof(message.msg_text), 1, 0);
        printf("Child received message: %s\n", message.msg_text);
        message.msg_type = 2;
        strcpy(message.msg_text, "Hello from child!");
        msgsnd(msgid, &message, sizeof(message.msg_text), 0);

        exit(0);
    } 
    else 
    {
        message.msg_type = 1;
        strcpy(message.msg_text, "Hello from parent!");
        msgsnd(msgid, &message, sizeof(message.msg_text), 0);
        printf("Parent sent message to child.\n");
        wait(NULL);
        msgrcv(msgid, &message, sizeof(message.msg_text), 2, 0);
        printf("Parent received message: %s\n", message.msg_text);
        msgctl(msgid, IPC_RMID, NULL);
    }

    return 0;
}

/* Write a C program to create a child process using fork() and demonstrate process synchronization using mutexes. */

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <pthread.h>
#include <sys/mman.h>
#include <sys/wait.h>

typedef struct 
{
    pthread_mutex_t mutex;
    int counter;
} shared_data_t;

int main() 
{
    shared_data_t *data = mmap(NULL, sizeof(shared_data_t),
                               PROT_READ | PROT_WRITE,
                               MAP_SHARED | MAP_ANONYMOUS, -1, 0);

    if (data == MAP_FAILED) 
    {
        perror("mmap");
        exit(EXIT_FAILURE);
    }
    pthread_mutexattr_t attr;
    pthread_mutexattr_init(&attr);
    pthread_mutexattr_setpshared(&attr, PTHREAD_PROCESS_SHARED);

    pthread_mutex_init(&data->mutex, &attr);
    data->counter = 0;

    pid_t pid = fork();

    if (pid < 0) 
    {
        perror("fork");
        exit(EXIT_FAILURE);
    }

    if (pid == 0) 
    {
        pthread_mutex_lock(&data->mutex);
        printf("Child: Entered critical section.\n");
        data->counter += 1;
        printf("Child: Counter = %d\n", data->counter);
        pthread_mutex_unlock(&data->mutex);
        exit(0);
    } 
    else 
    {
        pthread_mutex_lock(&data->mutex);
        printf("Parent: Entered critical section.\n");
        data->counter += 1;
        printf("Parent: Counter = %d\n", data->counter);
        pthread_mutex_unlock(&data->mutex);
        wait(NULL); 
        printf("Final Counter Value: %d\n", data->counter);
        pthread_mutex_destroy(&data->mutex);
        pthread_mutexattr_destroy(&attr);
        munmap(data, sizeof(shared_data_t));
    }

    return 0;
}

/*  Write a program in C to create a child process using fork() and print its PID. */

#include<stdio.h>
#include<unistd.h>
#include<sys/types.h>
int main()
{
        int pid;
	pid=fork();
	if(pid<0)
	{
		perror("fork failed");
		return 1;
	}
	else if(pid==0)
	{
		printf("hello from child process\n");	
		printf("Child PID: %d\n", getpid());
        	printf("Parent PID from child: %d\n", getppid());
	}
	else
	{
		printf("hello from the parent process\n");
		printf("Child PID: %d\n", getpid());
        	printf("Parent PID from child: %d\n", pid);
	}
	return 0;
}
/* Write a C program to demonstrate the use of the waitpid() function for process synchronization. */


#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int main() 
{
    int child1, child2,status;
    child1 = fork();

    if (child1 < 0) 
    {
        perror("fork failed for child1");
        exit(1);
    }

    if (child1 == 0) 
    {
        printf("Child 1 (PID: %d) is running...\n", getpid());
        sleep(2);
        printf("Child 1 (PID: %d) completed.\n", getpid());
        exit(1);
    }
    child2 = fork();

    if (child2 < 0) 
    {
        perror("fork failed for child2");
        exit(1);
    }

    if (child2 == 0) 
    {
        printf("Child 2 (PID: %d) is running...\n", getpid());
        sleep(4);
        printf("Child 2 (PID: %d) completed.\n", getpid());
        exit(2);
    }

    int wpid;
    wpid = waitpid(child1, &status, 0);
    if (wpid > 0) 
    {
        printf("Parent: Child 1 with PID %d exited with status %d.\n", wpid, WEXITSTATUS(status));
    }

    wpid = waitpid(child2, &status, 0);
    if (wpid > 0) 
    {
        printf("Parent: Child 2 with PID %d exited with status %d.\n", wpid, WEXITSTATUS(status));
    }

    printf("Parent: All child processes have completed.\n");

    return 0;
}

/* Write a C program to create a process using fork() and pass arguments to the child process. */

#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<sys/types.h>
#include<sys/wait.h>
int main()
{
	int pid;
	pid=fork();
	if(pid<0)
	{
		perror("fork failed");
		exit(EXIT_FAILURE);
	}
	else if(pid==0)
	{
		printf("child process (PID: %d) executing 'ls -l/'...\n",getpid());
		char *args[]={"ls","-l","/",NULL};
		execvp(args[0],args);
		perror("execvp failed");
		exit(EXIT_FAILURE);
	}
	else
	{
		printf("parent process (PID: %d) waiting for child...\n",getpid());
		wait(NULL);
		printf("parent process: child completed.\n");
	}
	return 0;
}
/* Write a C program to demonstrate the use of fork() system call. */

#include<stdio.h>
#include<unistd.h>
#include<sys/types.h>
int main()
{
        int pid;
	pid=fork();
	if(pid<0)
	{
		perror("fork failed");
		return 1;
	}
	else if(pid==0)
	{
		printf("hello from child process\n");
	}
	else
	{
		printf("hello from the parent process\n");
	}
	return 0;
}

/* Write a C program to create a child process using vfork() and demonstrate its usage. */

#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
int main()
{
	int v=10,pid;
	printf("before vfork. PID: %d\n",getpid());
	pid=vfork();
	if(pid<0)
	{
		perror("vfork failed");
		exit(EXIT_FAILURE);
	}
	else if(pid==0)
	{
		printf("child process: PID=%d,parent PID=%d\n",getpid(),getppid());
		v +=5;
		printf("child process changed variable to %d\n",v);
		exit(0);
	}
	else
	{
		printf("parent process: PID=%d\n",getpid());
		printf("parent process sees variable=%d\n",v);
	}
	return 0;
}
/* Write a C program to demonstrate the use of the clone() system call to create a thread */

#include<sched.h>
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<sys/wait.h>
#include<linux/sched.h>
#define _GNU_SOURCE
#define STACK_SIZE 1024*1024

int thread_function(void *arg)
{
	printf("inside child thread: PID=%d, PPID=%d\n",getpid(),getppid());
	return 0;
}
int main()
{
	void *stack=malloc(STACK_SIZE);
	if(!stack)
	{
		perror("malloc");
		exit(1);
	}
	printf("inside parent process: PID=%d\n",getpid());
	int flags=CLONE_VM | CLONE_FS | CLONE_FILES | CLONE_SIGHAND | SIGCHLD;
	int child_pid=clone(thread_function,stack+STACK_SIZE,flags,NULL);
	if(child_pid==-1)
	{
		perror("clone");
		free(stack);
		exit(1);
	}
	waitpid(child_pid,NULL,0);
	printf("child thread exited. parent exiting.\n");
	free(stack);
	return 0;
}
/* Write a C program to create a process using fork() and change its scheduling policy using sched_setscheduler(). */

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sched.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <errno.h>
#include <string.h>

int main() 
{
    pid_t pid;
    struct sched_param schedParam;
    pid = fork();
    if (pid < 0) 
    {
        perror("fork failed");
        exit(1);
    }

    if (pid == 0) 
    {
        printf("Child PID: %d\n", getpid());
        schedParam.sched_priority = 10;
        if (sched_setscheduler(0, SCHED_FIFO, &schedParam) == -1) 
	{
            perror("sched_setscheduler failed");
            exit(1);
        }

        int policy = sched_getscheduler(0);
        printf("Child process scheduling policy changed to: ");

        switch (policy) 
	{
            case SCHED_OTHER:
                printf("SCHED_OTHER\n"); break;
            case SCHED_FIFO:
                printf("SCHED_FIFO\n"); break;
            case SCHED_RR:
                printf("SCHED_RR\n"); break;
            default:
                printf("UNKNOWN\n");
        }

        for (int i = 0; i < 5; i++) 
	{
            printf("Child running iteration %d\n", i + 1);
            sleep(1);
        }

        exit(0);
    } 
    else 
    {
        wait(NULL);
        printf("Parent process: child has finished.\n");
    }

    return 0;
}

/* Write a C program to demonstrate the use of the prctl() system call to change process attributes. */

#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <sys/prctl.h>
#include <string.h>
#include <unistd.h>

int main() 
{
    char name[17];
    if (prctl(PR_SET_NAME, "MyProcess", 0, 0, 0) == -1) 
    {
        perror("prctl - PR_SET_NAME");
        exit(EXIT_FAILURE);
    }

    if (prctl(PR_GET_NAME, name, 0, 0, 0) == -1) 
    {
        perror("prctl - PR_GET_NAME");
        exit(EXIT_FAILURE);
    }

    printf("Current process name: %s\n", name);

    printf("Sleeping for 10 seconds... check with `ps -C MyProcess`\n");
    sleep(10);

    return 0;
}
/* Write a C program to create a child process using fork() and demonstrate process synchronization using condition variables. */

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <pthread.h>
#include <sys/mman.h>
#include <sys/wait.h>
#include <errno.h>
typedef struct 
{
    pthread_mutex_t mutex;
    pthread_cond_t cond;
    int ready;
} shared_data_t;

int main() 
{
    shared_data_t *data = mmap(NULL, sizeof(shared_data_t),
                               PROT_READ | PROT_WRITE,
                               MAP_SHARED | MAP_ANONYMOUS, -1, 0);

    if (data == MAP_FAILED) 
    {
        perror("mmap");
        exit(EXIT_FAILURE);
    }
    pthread_mutexattr_t mutex_attr;
    pthread_condattr_t cond_attr;

    pthread_mutexattr_init(&mutex_attr);
    pthread_mutexattr_setpshared(&mutex_attr, PTHREAD_PROCESS_SHARED);

    pthread_condattr_init(&cond_attr);
    pthread_condattr_setpshared(&cond_attr, PTHREAD_PROCESS_SHARED);

    pthread_mutex_init(&data->mutex, &mutex_attr);
    pthread_cond_init(&data->cond, &cond_attr);

    data->ready = 0;

    pid_t pid = fork();

    if (pid < 0) 
    {
        perror("fork");
        exit(EXIT_FAILURE);
    }

    if (pid == 0) 
    {
        sleep(2);
        pthread_mutex_lock(&data->mutex);
        printf("Child: signaling condition variable.\n");
        data->ready = 1;
        pthread_cond_signal(&data->cond);
        pthread_mutex_unlock(&data->mutex);
        exit(0);
    } 
    else 
    {
        pthread_mutex_lock(&data->mutex);
        while (data->ready == 0) 
	{
            printf("Parent: waiting for condition variable...\n");
            pthread_cond_wait(&data->cond, &data->mutex);
        }
        printf("Parent: received signal, proceeding.\n");
        pthread_mutex_unlock(&data->mutex);
        wait(NULL);
        pthread_mutex_destroy(&data->mutex);
        pthread_cond_destroy(&data->cond);
        pthread_mutexattr_destroy(&mutex_attr);
        pthread_condattr_destroy(&cond_attr);
        munmap(data, sizeof(shared_data_t));
    }
    return 0;
}

/* Write a C program to create a child process using fork() and demonstrate process communication using named pipes (FIFOs). */

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <string.h>
#include <sys/wait.h>

#define FIFO_PATH "/tmp/myfifo"

int main() 
{
    char buf[100];
    int fd;
    if (mkfifo(FIFO_PATH, 0666) == -1) 
    {
        perror("mkfifo");
    }
    pid_t pid = fork();
    if (pid < 0) 
    {
        perror("fork");
        unlink(FIFO_PATH);
        exit(EXIT_FAILURE);
    }
    if (pid == 0) 
    {
        fd = open(FIFO_PATH, O_RDONLY);
        if (fd == -1) 
	{
            perror("child open read");
            exit(EXIT_FAILURE);
        }
        read(fd, buf, sizeof(buf));
        printf("Child received: %s\n", buf);
        close(fd);
        fd = open(FIFO_PATH, O_WRONLY);
        if (fd == -1) 
	{
            perror("child open write");
            exit(EXIT_FAILURE);
        }
        const char *child_msg = "Hello from child!";
        write(fd, child_msg, strlen(child_msg) + 1);
        close(fd);
        exit(0);
    } 
    else 
    {
        fd = open(FIFO_PATH, O_WRONLY);
        if (fd == -1) 
	{
            perror("parent open write");
            unlink(FIFO_PATH);
            exit(EXIT_FAILURE);
        }
        const char *parent_msg = "Hello from parent!";
        write(fd, parent_msg, strlen(parent_msg) + 1);
        close(fd);
        fd = open(FIFO_PATH, O_RDONLY);
        if (fd == -1) 
	{
            perror("parent open read");
            unlink(FIFO_PATH);
            exit(EXIT_FAILURE);
        }
        read(fd, buf, sizeof(buf));
        printf("Parent received: %s\n", buf);
        close(fd);
        wait(NULL);
        unlink(FIFO_PATH);
    }

    return 0;
}
/* Write a C program to create multiple child processes using fork() and display their PIDs */


#include<stdio.h>
#include<unistd.h>
#include<sys/types.h>
#include<sys/wait.h>
#include <stdlib.h>
#define N_C 4
int main()
{
        int pid;
	for(int i=0;i<N_C;i++)
	{
		pid=fork();
		if(pid<0)
		{
			perror("fork failed");
			return 1;
		}
		else if(pid==0)
		{
			printf("hello from child process\n");
			printf("Child %d: My PID is %d, My Parent PID is %d\n", i + 1, getpid(), getppid());
            		exit(0);
		}
	}
	for (int i = 0; i < N_C; i++) 
	{
        	wait(NULL);
	}
	printf("Parent process (PID: %d) has created %d child processes.\n", getpid(), N_C);
	return 0;
}
/* Write a program in C to create a daemon process. */

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

void create_daemon() 
{
    int pid;

    pid = fork();

    if (pid < 0) 
    {
        perror("fork failed");
        exit(EXIT_FAILURE);
    }

    if (pid > 0) 
    {
        printf("Daemon PID: %d\n", pid);
        exit(EXIT_SUCCESS);
    }

    if (setsid() < 0) 
    {
        perror("setsid failed");
        exit(EXIT_FAILURE);
    }
    pid = fork();
    if (pid < 0) 
    {
        perror("Second fork failed");
        exit(EXIT_FAILURE);
    }
    if (pid > 0) 
    {
        exit(EXIT_SUCCESS);
    }
    umask(0);

    chdir("/");

    close(STDIN_FILENO);
    close(STDOUT_FILENO);
    close(STDERR_FILENO);
    open("/dev/null", O_RDONLY);
    open("/dev/null", O_WRONLY);
    open("/dev/null", O_RDWR);
    while (1) 
    {
        sleep(10);
    }
}

int main() 
{
    create_daemon();
    return 0;
}
/*Write a program in C to demonstrate process synchronization using semaphores. */

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/ipc.h>
#include <sys/sem.h>
#include <sys/wait.h>

union semun 
{
    int val;            
    struct semid_ds *buf; 
    unsigned short *array;
};

void sem_wait(int semid) 
{
    struct sembuf sb = {0, -1, 0}; 
    if (semop(semid, &sb, 1) == -1) 
    {
        perror("semop wait failed");
        exit(1);
    }
}

void sem_signal(int semid) 
{
    struct sembuf sb = {0, 1, 0}; 
    if (semop(semid, &sb, 1) == -1) 
    {
        perror("semop signal failed");
        exit(1);
    }
}

int main() 
{
    key_t key = ftok("semfile", 65);
    int semid = semget(key, 1, 0666 | IPC_CREAT);

    if (semid == -1) 
    {
        perror("semget failed");
        exit(1);
    }

    union semun u;
    u.val = 1;
    if (semctl(semid, 0, SETVAL, u) == -1) 
    {
        perror("semctl SETVAL failed");
        exit(1);
    }

    pid_t pid = fork();

    if (pid < 0) 
    {
        perror("fork failed");
        exit(1);
    } 
    else if (pid == 0) 
    {
        sem_wait(semid);
        printf("Child process (PID: %d) entering critical section.\n", getpid());
        sleep(2);
        printf("Child process (PID: %d) leaving critical section.\n", getpid());
        sem_signal(semid);
        exit(0);
    } 
    else 
    {
        sem_wait(semid);
        printf("Parent process (PID: %d) entering critical section.\n", getpid());
        sleep(2);
        printf("Parent process (PID: %d) leaving critical section.\n", getpid());
        sem_signal(semid);
        wait(NULL);
        semctl(semid, 0, IPC_RMID);
    }

    return 0;
}
/* Write a C program to create a process group and change its process group ID (PGID). */

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <errno.h>
#include <sys/wait.h>

int main() 
{
    pid_t pid;
    pid = fork();

    if (pid < 0) 
    {
        perror("fork failed");
        exit(EXIT_FAILURE);
    }

    if (pid == 0) 
    {
        printf("Child Process:\n");
        printf("PID: %d\n", getpid());
        printf("Old PGID: %d\n", getpgrp());
        if (setpgid(0, 0) == -1) 
	{
            perror("setpgid failed in child");
            exit(EXIT_FAILURE);
        }
	printf("New PGID: %d\n", getpgrp());
        sleep(10);
        printf("Child exiting...\n");
    } 
    else 
    {
        printf("Parent Process:\n");
        printf("PID: %d\n", getpid());
        printf("PGID: %d\n", getpgrp());
        wait(NULL);
        printf("Parent exiting...\n");
    }

    return 0;
}
/* Write a C program to create a pipeline between two processes using the pipe() system call. */

#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<string.h>
int main()
{
	int fd[2],pid;
	char message[]="hello from parent to child!";
	char buffer[100];
	if(pipe(fd)== -1)
	{
		perror("pipe failed");
		exit(EXIT_FAILURE);
	}
	pid=fork();
	if(pid<0)
	{
		perror("fork failed");
		exit(EXIT_FAILURE);
	}
	else if(pid==0)
	{
		close(fd[1]);
		read(fd[0],buffer,sizeof(buffer));
		printf("child received: %s\n",buffer);
		close(fd[0]);
	}
	else
	{
		close(fd[0]);
		write(fd[1],message,strlen(message)+1);
		close(fd[1]);
	}
	return 0;
}
/* Write a C program to create a child process using fork() and communicate between parent and child using pipes. */

#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <sys/wait.h>

int main() 
{
    int pipe1[2];
    int pipe2[2];
    int pid;

    if (pipe(pipe1) == -1 || pipe(pipe2) == -1) 
    {
        perror("Pipe failed");
        return 1;
    }

    pid = fork();

    if (pid < 0) 
    {
        perror("Fork failed");
        return 1;
    }

    if (pid > 0) 
    {
        close(pipe1[0]);
        close(pipe2[1]);

        char parentMsg[] = "Hello from parent!";
        write(pipe1[1], parentMsg, strlen(parentMsg) + 1);
        printf("Parent sent: %s\n", parentMsg);

        char childReply[100];
        read(pipe2[0], childReply, sizeof(childReply));
        printf("Parent received: %s\n", childReply);

        close(pipe1[1]);
        close(pipe2[0]);

        wait(NULL);
    } 
    else 
    {
        close(pipe1[1]);
        close(pipe2[0]);

        char buffer[100];
        read(pipe1[0], buffer, sizeof(buffer));
        printf("Child received: %s\n", buffer);

        char childMsg[] = "Hello from child!";
        write(pipe2[1], childMsg, strlen(childMsg) + 1);

        close(pipe1[0]);
        close(pipe2[1]);
    }

    return 0;
}

/* Write a C program to illustrate the use of the execvp() function. */

#include<stdio.h>
#include<unistd.h>
#include<stdlib.h>
#include<sys/wait.h>
int main()
{
	int pid;
	pid=fork();
	if(pid<0)
	{
		perror("fork failed");
		return 1;
	}
	else if(pid==0)
	{
		printf("child process\n");
		char *args[]={"ls","-l",NULL};
		execvp(args[0],args);
		perror("execvp failed");
		exit(1);
	}
	else
	{
		printf("parent process: waiting for child\n");
		wait(NULL);
		printf("parent process: child complete\n");
	}
	return 0;
}

/* Write a C program to create a child process using fork() and demonstrate process synchronization using semaphores. */

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/ipc.h>
#include <sys/sem.h>
#include <sys/types.h>
#include <sys/wait.h>

union semun 
{
    int val;             
    struct semid_ds *buf; 
    unsigned short *array;
};
void sem_wait(int semid) 
{
    struct sembuf sb = {0, -1, 0};
    semop(semid, &sb, 1);
}
void sem_signal(int semid) 
{
    struct sembuf sb = {0, 1, 0};
    semop(semid, &sb, 1);
}

int main() 
{
    key_t key = ftok("file.txt", 65); 
    int semid;
    union semun sem_union;
    semid = semget(key, 1, 0666 | IPC_CREAT);
    if (semid < 0) 
    {
        perror("semget");
        exit(1);
    }

    sem_union.val = 0;
    if (semctl(semid, 0, SETVAL, sem_union) == -1) 
    {
        perror("semctl - SETVAL");
        exit(1);
    }

    pid_t pid = fork();
    if (pid < 0) 
    {
        perror("fork");
        exit(1);
    }
    if (pid == 0) 
    {
        printf("Child: doing some work...\n");
        sleep(2); 
        printf("Child: signaling semaphore\n");
        sem_signal(semid);
        exit(0);
    } 
    else 
    {
        printf("Parent: waiting for child to signal...\n");
        sem_wait(semid);
        printf("Parent: received signal, proceeding.\n");
        wait(NULL);
        if (semctl(semid, 0, IPC_RMID, sem_union) == -1) 
	{
            perror("semctl - IPC_RMID");
            exit(1);
        }
    }

    return 0;
}

/* Write a C program to create a child process using fork() and demonstrate process communication using sockets. */

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/socket.h>
#include <string.h>
#include <sys/wait.h>

int main() 
{
    int sockfd[2];
    pid_t pid;

    if (socketpair(AF_UNIX, SOCK_STREAM, 0, sockfd) == -1) 
    {
        perror("socketpair");
        exit(EXIT_FAILURE);
    }

    pid = fork();
    if (pid < 0) 
    {
        perror("fork");
        exit(EXIT_FAILURE);
    }

    if (pid == 0) 
    {
        close(sockfd[0]);

        char buffer[100];
        read(sockfd[1], buffer, sizeof(buffer));
        printf("Child received: %s\n", buffer);

        const char *child_msg = "Hello from child!";
        write(sockfd[1], child_msg, strlen(child_msg) + 1);

        close(sockfd[1]);
        exit(0);
    } 
    else 
    {
        close(sockfd[1]);

        const char *parent_msg = "Hello from parent!";
        write(sockfd[0], parent_msg, strlen(parent_msg) + 1);

        char buffer[100];
        read(sockfd[0], buffer, sizeof(buffer));
        printf("Parent received: %s\n", buffer);

        close(sockfd[0]);
        wait(NULL); 
    }

    return 0;
}
