1. Write a C program to create a thread that prints "Hello, World!"? 
```
#include <stdio.h>
#include <pthread.h>

void* print_hello(void* arg) 
{
    printf("Hello, World!\n");
    return NULL;
}

int main() 
{
    pthread_t thread;
    if (pthread_create(&thread, NULL, print_hello, NULL) != 0) 
    {
        perror("pthread_create");
        return 1;
    }
    pthread_join(thread, NULL);

    return 0;
}
````
2. Modify the above program to create multiple threads, each printing its own message? 
```
#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>

#define NUM_THREADS 5

void* print_message(void* arg) 
{
    int thread_num = *(int*)arg;
    printf("Hello from thread %d\n", thread_num);
    free(arg); 
    pthread_exit(NULL);
}

int main() 
{
    pthread_t threads[NUM_THREADS];

    for (int i = 0; i < NUM_THREADS; i++) 
    {
        int* thread_id = malloc(sizeof(int));
        if (thread_id == NULL) 
	{
            perror("malloc");
            exit(EXIT_FAILURE);
        }
        *thread_id = i + 1;
        if (pthread_create(&threads[i], NULL, print_message, thread_id) != 0) 
	{
            perror("pthread_create");
            free(thread_id);
            exit(EXIT_FAILURE);
        }
    }
    for (int i = 0; i < NUM_THREADS; i++) 
    {
        pthread_join(threads[i], NULL);
    }

    return 0;
}
```
3. Develop a C program to create two threads that print numbers from 1 to 10 concurrently 
```
#include <stdio.h>
#include <pthread.h>
#include <unistd.h> 

void* print_numbers(void* arg) 
{
    int thread_id = *(int*)arg;

    for (int i = 1; i <= 10; i++) 
    {
        printf("Thread %d: %d\n", thread_id, i);
        usleep(100000);
    }

    return NULL;
}

int main() 
{
    pthread_t thread1, thread2;
    int id1 = 1, id2 = 2;

    pthread_create(&thread1, NULL, print_numbers, &id1);
    pthread_create(&thread2, NULL, print_numbers, &id2);

    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);

    return 0;
}
```
4. Implement a C program to create a thread that calculates the factorial of a given number?
```
#include <stdio.h>
#include <pthread.h>

typedef struct 
{
    int number;
    unsigned long long result;
} factorial_data_t;

void* factorial_thread(void* arg) 
{
    factorial_data_t* data = (factorial_data_t*)arg;
    int n = data->number;
    unsigned long long fact = 1;

    for (int i = 1; i <= n; i++) 
    {
        fact *= i;
    }

    data->result = fact;
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    factorial_data_t data;

    printf("Enter a number: ");
    scanf("%d", &data.number);

    if (data.number < 0) 
    {
        printf("Factorial is not defined for negative numbers.\n");
        return 1;
    }

    if (pthread_create(&thread, NULL, factorial_thread, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }
    pthread_join(thread, NULL);

    printf("Factorial of %d is %llu\n", data.number, data.result);

    return 0;
}
5. Write a C program to create two threads that print their thread IDs?
```
#include <stdio.h>
#include <pthread.h>

void* print_thread_id(void* arg) 
{
    pthread_t tid = pthread_self();
    printf("Thread ID: %lu\n", (unsigned long)tid);
    return NULL;
}

int main() 
{
    pthread_t thread1, thread2;
    pthread_create(&thread1, NULL, print_thread_id, NULL);
    pthread_create(&thread2, NULL, print_thread_id, NULL);
    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);

    return 0;
}
```
6. Develop a C program to create a thread that prints the sum of two numbers?
```
#include <stdio.h>
#include <pthread.h>

typedef struct 
{
    int a;
    int b;
} sum_data_t;

void* calculate_sum(void* arg) 
{
    sum_data_t* data = (sum_data_t*)arg;
    int sum = data->a + data->b;
    printf("Sum of %d and %d is %d\n", data->a, data->b, sum);
    return NULL;
}

int main() 
{
    pthread_t thread;
    sum_data_t data;

    printf("Enter two numbers: ");
    scanf("%d %d", &data.a, &data.b);

    if (pthread_create(&thread, NULL, calculate_sum, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    return 0;
}
```
7. Implement a C program to create a thread that calculates the square of a number? 
```
#include <stdio.h>
#include <pthread.h>

typedef struct 
{
    int number;
    int result;
} square_data_t;

void* calculate_square(void* arg) 
{
    square_data_t* data = (square_data_t*)arg;
    data->result = data->number * data->number;
    printf("Square of %d is %d\n", data->number, data->result);
    return NULL;
}

int main() 
{
    pthread_t thread;
    square_data_t data;

    printf("Enter a number: ");
    scanf("%d", &data.number);

    if (pthread_create(&thread, NULL, calculate_square, &data) != 0) {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    return 0;
}
```
8. Write a C program to create a thread that prints the current date and time? 
```
#include <stdio.h>
#include <pthread.h>
#include <time.h>

void* print_date_time(void* arg) 
{
    time_t now = time(NULL);
    if (now == (time_t)(-1)) 
    {
        perror("time");
        pthread_exit(NULL);
    }

    struct tm *local_time = localtime(&now);
    if (local_time == NULL) 
    {
        perror("localtime");
        pthread_exit(NULL);
    }

    char buffer[100];
    if (strftime(buffer, sizeof(buffer), "%Y-%m-%d %H:%M:%S", local_time) == 0) 
    {
        fprintf(stderr, "strftime returned 0");
        pthread_exit(NULL);
    }

    printf("Current date and time: %s\n", buffer);
    return NULL;
}

int main() 
{
    pthread_t thread;

    if (pthread_create(&thread, NULL, print_date_time, NULL) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);
    return 0;
}
```
9. Develop a C program to create a thread that checks if a number is prime? 
```
#include <stdio.h>
#include <pthread.h>
#include <math.h>

typedef struct 
{
    int number;
    int is_prime;
} prime_data_t;

void* check_prime(void* arg) 
{
    prime_data_t* data = (prime_data_t*)arg;
    int n = data->number;

    if (n <= 1) 
    {
        data->is_prime = 0;
        pthread_exit(NULL);
    }

    for (int i = 2; i <= (int)sqrt(n); i++) 
    {
        if (n % i == 0) 
	{
            data->is_prime = 0;
            pthread_exit(NULL);
        }
    }

    data->is_prime = 1;
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    prime_data_t data;

    printf("Enter a number: ");
    scanf("%d", &data.number);

    if (pthread_create(&thread, NULL, check_prime, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    if (data.is_prime)
        printf("%d is a prime number.\n", data.number);
    else
        printf("%d is not a prime number.\n", data.number);

    return 0;
}
```
10. Implement a C program to create a thread that checks if a given string is a palindrome?
```
#include <stdio.h>
#include <string.h>
#include <pthread.h>
#include <ctype.h>

typedef struct 
{
    char str[100];
    int is_palindrome;
} palindrome_data_t;

void* check_palindrome(void* arg) 
{
    palindrome_data_t* data = (palindrome_data_t*)arg;
    char* s = data->str;

    int left = 0;
    int right = strlen(s) - 1;

    while (left < right) 
    {
        char left_char = tolower(s[left]);
        char right_char = tolower(s[right]);

        if (left_char != right_char) 
	{
            data->is_palindrome = 0;
            pthread_exit(NULL);
        }
        left++;
        right--;
    }

    data->is_palindrome = 1;
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    palindrome_data_t data;

    printf("Enter a string: ");
    fgets(data.str, sizeof(data.str), stdin);
    
    data.str[strcspn(data.str, "\n")] = '\0';

    if (pthread_create(&thread, NULL, check_palindrome, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    if (data.is_palindrome)
        printf("\"%s\" is a palindrome.\n", data.str);
    else
        printf("\"%s\" is not a palindrome.\n", data.str);

    return 0;
}

/* Write a C program to create a thread that prints "Hello, World!" with thread
synchronization? */

#include <stdio.h>
#include <pthread.h>

pthread_mutex_t print_mutex;

void* print_hello(void* arg) 
{
    pthread_mutex_lock(&print_mutex);

    printf("Hello, World!\n");

    pthread_mutex_unlock(&print_mutex);
    return NULL;
}

int main() 
{
    pthread_t thread;

    if (pthread_mutex_init(&print_mutex, NULL) != 0) 
    {
        perror("pthread_mutex_init");
        return 1;
    }
    if (pthread_create(&thread, NULL, print_hello, NULL) != 0) 
    {
        perror("pthread_create");
        pthread_mutex_destroy(&print_mutex);
        return 1;
    }

    pthread_join(thread, NULL);
    pthread_mutex_destroy(&print_mutex);

    return 0;
}

/* Develop a C program to create two threads that print their thread IDs and synchronize their output? */

#include <stdio.h>
#include <pthread.h>

pthread_mutex_t mutex;
pthread_cond_t cond;

int turn = 1; 

void* thread_func(void* arg) 
{
    int thread_num = *(int*)arg;
    pthread_mutex_lock(&mutex);
    while (turn != thread_num) 
    {
        pthread_cond_wait(&cond, &mutex);
    }
    printf("Thread %d ID: %lu\n", thread_num, (unsigned long)pthread_self());
    turn = (thread_num == 1) ? 2 : 1;
    pthread_cond_signal(&cond);

    pthread_mutex_unlock(&mutex);

    return NULL;
}

int main() 
{
    pthread_t thread1, thread2;
    int t1 = 1, t2 = 2;

    pthread_mutex_init(&mutex, NULL);
    pthread_cond_init(&cond, NULL);

    pthread_create(&thread1, NULL, thread_func, &t1);
    pthread_create(&thread2, NULL, thread_func, &t2);

    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);

    pthread_mutex_destroy(&mutex);
    pthread_cond_destroy(&cond);

    return 0;
}

/* Write a C program to create a thread that performs addition of two numbers with mutex locks? */

#include <stdio.h>
#include <pthread.h>
typedef struct 
{
    int a;
    int b;
    int result;
    pthread_mutex_t *mutex;
} addition_data_t;

void* add_numbers(void* arg) 
{
    addition_data_t* data = (addition_data_t*)arg;

    pthread_mutex_lock(data->mutex);
    data->result = data->a + data->b;
    printf("Sum of %d and %d is %d\n", data->a, data->b, data->result);
    pthread_mutex_unlock(data->mutex);

    return NULL;
}

int main() 
{
    pthread_t thread;
    pthread_mutex_t mutex;
    addition_data_t data;

    printf("Enter two numbers: ");
    scanf("%d %d", &data.a, &data.b);
    if (pthread_mutex_init(&mutex, NULL) != 0) 
    {
        perror("Mutex init failed");
        return 1;
    }
    data.mutex = &mutex;
    if (pthread_create(&thread, NULL, add_numbers, &data) != 0) 
    {
        perror("pthread_create");
        pthread_mutex_destroy(&mutex);
        return 1;
    }
    pthread_join(thread, NULL);

    pthread_mutex_destroy(&mutex);

    return 0;
}

/* Implement a C program to create two threads that increment and decrement a shared variable, respectively, using mutex locks? */

#include <stdio.h>
#include <pthread.h>

#define NUM_ITERATIONS 100000

int shared_var = 0;
pthread_mutex_t mutex;

void* increment_thread(void* arg) 
{
    for (int i = 0; i < NUM_ITERATIONS; i++) 
    {
        pthread_mutex_lock(&mutex);
        shared_var++;
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

void* decrement_thread(void* arg) 
{
    for (int i = 0; i < NUM_ITERATIONS; i++) 
    {
        pthread_mutex_lock(&mutex);
        shared_var--;
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

int main() 
{
    pthread_t inc_thread, dec_thread;

    if (pthread_mutex_init(&mutex, NULL) != 0) 
    {
        perror("Mutex init failed");
        return 1;
    }

    pthread_create(&inc_thread, NULL, increment_thread, NULL);
    pthread_create(&dec_thread, NULL, decrement_thread, NULL);

    pthread_join(inc_thread, NULL);
    pthread_join(dec_thread, NULL);

    printf("Final value of shared_var: %d\n", shared_var);

    pthread_mutex_destroy(&mutex);

    return 0;
}

/* Develop a C program to create a thread that reads input from the user and synchronizes access to shared resources? */

#include <stdio.h>
#include <pthread.h>
#include <string.h>

#define BUFFER_SIZE 100

char input[BUFFER_SIZE];
pthread_mutex_t input_mutex;

void* read_input(void* arg) 
{
    pthread_mutex_lock(&input_mutex);

    printf("Enter a string: ");
    fgets(input, BUFFER_SIZE, stdin);
    input[strcspn(input, "\n")] = '\0';

    pthread_mutex_unlock(&input_mutex);

    return NULL;
}

int main() 
{
    pthread_t thread;
    if (pthread_mutex_init(&input_mutex, NULL) != 0) 
    {
        perror("Mutex init failed");
        return 1;
    }

    if (pthread_create(&thread, NULL, read_input, NULL) != 0) 
    {
        perror("pthread_create");
        pthread_mutex_destroy(&input_mutex);
        return 1;
    }
    pthread_join(thread, NULL);
    pthread_mutex_lock(&input_mutex);
    printf("You entered: \"%s\"\n", input);
    pthread_mutex_unlock(&input_mutex);

    pthread_mutex_destroy(&input_mutex);

    return 0;
}

/* Implement a C program to create a thread that prints prime numbers up to a given limit with mutex locks? */

#include <stdio.h>
#include <pthread.h>
#include <math.h>

pthread_mutex_t print_mutex;

int is_prime(int n) 
{
    if (n <= 1) return 0;
    if (n == 2) return 1;

    for (int i = 2; i <= (int)sqrt(n); i++) 
    {
        if (n % i == 0)
            return 0;
    }
    return 1;
}

typedef struct 
{
    int limit;
} thread_data_t;

void* print_primes(void* arg) 
{
    thread_data_t* data = (thread_data_t*)arg;
    int limit = data->limit;

    for (int i = 2; i <= limit; i++) 
    {
        if (is_prime(i)) 
	{
            pthread_mutex_lock(&print_mutex);
            printf("%d is a prime number\n", i);
            pthread_mutex_unlock(&print_mutex);
        }
    }

    return NULL;
}

int main() 
{
    pthread_t thread;
    thread_data_t data;

    printf("Enter the upper limit: ");
    scanf("%d", &data.limit);
    if (pthread_mutex_init(&print_mutex, NULL) != 0) 
    {
        perror("Mutex init failed");
        return 1;
    }
    if (pthread_create(&thread, NULL, print_primes, &data) != 0) 
    {
        perror("pthread_create");
        pthread_mutex_destroy(&print_mutex);
        return 1;
    }
    pthread_join(thread, NULL);
    pthread_mutex_destroy(&print_mutex);

    return 0;
}

/* Implement a C program to create a thread that calculates the sum of Fibonacci numbers upto a given limit using dynamic programming with mutex locks? */

#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>

pthread_mutex_t sum_mutex;
unsigned long long fib_sum = 0;

typedef struct 
{
    int limit;
} fib_data_t;

void* compute_fibonacci_sum(void* arg) 
{
    fib_data_t* data = (fib_data_t*)arg;
    int n = data->limit;

    if (n < 0) 
    {
        pthread_exit(NULL);
    }

    unsigned long long* fib = malloc((n + 1) * sizeof(unsigned long long));
    if (fib == NULL) 
    {
        perror("malloc");
        pthread_exit(NULL);
    }

    fib[0] = 0;
    if (n >= 1)
        fib[1] = 1;

    for (int i = 2; i <= n; i++) 
    {
        fib[i] = fib[i - 1] + fib[i - 2];
    }

    unsigned long long local_sum = 0;
    for (int i = 0; i <= n; i++) 
    {
        local_sum += fib[i];
    }

    pthread_mutex_lock(&sum_mutex);
    fib_sum = local_sum;
    pthread_mutex_unlock(&sum_mutex);

    free(fib);
    return NULL;
}

int main() 
{
    pthread_t thread;
    fib_data_t data;

    printf("Enter the limit (n): ");
    scanf("%d", &data.limit);

    if (pthread_mutex_init(&sum_mutex, NULL) != 0) 
    {
        perror("Mutex init failed");
        return 1;
    }
    if (pthread_create(&thread, NULL, compute_fibonacci_sum, &data) != 0) 
    {
        perror("pthread_create");
        pthread_mutex_destroy(&sum_mutex);
        return 1;
    }
    pthread_join(thread, NULL);

    pthread_mutex_lock(&sum_mutex);
    printf("Sum of Fibonacci numbers up to F(%d): %llu\n", data.limit, fib_sum);
    pthread_mutex_unlock(&sum_mutex);

    pthread_mutex_destroy(&sum_mutex);
    return 0;
}

/* Write a C program to create a thread that checks if a given year is a leap year using dynamic programming with mutex locks? */

#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>

#define MAX_YEARS 10000

pthread_mutex_t mutex;
typedef struct 
{
    int year;
    int is_cached;
    int is_leap;
} leap_cache_t;

leap_cache_t year_cache[MAX_YEARS] = {0};

typedef struct 
{
    int year;
} thread_data_t;

void* check_leap_year(void* arg) 
{
    thread_data_t* data = (thread_data_t*)arg;
    int year = data->year;

    pthread_mutex_lock(&mutex);

    if (year < 0 || year >= MAX_YEARS) {
        printf("Year %d is out of supported range (0 to %d).\n", year, MAX_YEARS - 1);
        pthread_mutex_unlock(&mutex);
        pthread_exit(NULL);
    }
    if (year_cache[year].is_cached) 
    {
        printf("Cached: Year %d is %sa leap year.\n", year,
               year_cache[year].is_leap ? "" : "not ");
    } 
    else 
    {
        int is_leap = 0;
        if ((year % 4 == 0 && year % 100 != 0) || (year % 400 == 0)) 
	{
            is_leap = 1;
        }
        year_cache[year].year = year;
        year_cache[year].is_leap = is_leap;
        year_cache[year].is_cached = 1;

        printf("Computed: Year %d is %sa leap year.\n", year,
               is_leap ? "" : "not ");
    }

    pthread_mutex_unlock(&mutex);
    return NULL;
}

int main() 
{
    pthread_t thread;
    thread_data_t data;

    printf("Enter a year: ");
    scanf("%d", &data.year);

    if (pthread_mutex_init(&mutex, NULL) != 0) 
    {
        perror("Mutex init failed");
        return 1;
    }

    if (pthread_create(&thread, NULL, check_leap_year, &data) != 0) 
    {
        perror("Thread creation failed");
        pthread_mutex_destroy(&mutex);
        return 1;
    }

    pthread_join(thread, NULL);
    pthread_mutex_destroy(&mutex);

    return 0;
}

/* Write a C program to create a thread that checks if a given string is a palindrome using dynamic programming with mutex locks? */
#include <stdio.h>
#include <string.h>
#include <pthread.h>
#include <stdlib.h>

#define MAX_CACHE 100

pthread_mutex_t mutex;

typedef struct 
{
    char str[100];
    int is_palindrome;
    int is_cached;
} palindrome_cache_t;

palindrome_cache_t cache[MAX_CACHE];
int cache_size = 0;

typedef struct 
{
    char input[100];
} thread_data_t;

int is_palindrome(const char* s) 
{
    int len = strlen(s);
    for (int i = 0; i < len / 2; i++) 
    {
        if (s[i] != s[len - i - 1])
            return 0;
    }
    return 1;
}

void* check_palindrome(void* arg) 
{
    thread_data_t* data = (thread_data_t*)arg;
    char* str = data->input;

    pthread_mutex_lock(&mutex);
    for (int i = 0; i < cache_size; i++) {
        if (strcmp(cache[i].str, str) == 0) {
            printf("Cached: \"%s\" is %sa palindrome.\n", str, cache[i].is_palindrome ? "" : "not ");
            pthread_mutex_unlock(&mutex);
            return NULL;
        }
    }
    int result = is_palindrome(str);
    if (cache_size < MAX_CACHE) 
    {
        strcpy(cache[cache_size].str, str);
        cache[cache_size].is_palindrome = result;
        cache[cache_size].is_cached = 1;
        cache_size++;
    }

    printf("Computed: \"%s\" is %sa palindrome.\n", str, result ? "" : "not ");

    pthread_mutex_unlock(&mutex);
    return NULL;
}

int main() 
{
    pthread_t thread;
    thread_data_t data;

    printf("Enter a string: ");
    fgets(data.input, sizeof(data.input), stdin);
    data.input[strcspn(data.input, "\n")] = '\0'; 

    if (pthread_mutex_init(&mutex, NULL) != 0) 
    {
        perror("Mutex init failed");
        return 1;
    }

    if (pthread_create(&thread, NULL, check_palindrome, &data) != 0) 
    {
        perror("Thread creation failed");
        pthread_mutex_destroy(&mutex);
        return 1;
    }

    pthread_join(thread, NULL);
    pthread_mutex_destroy(&mutex);

    return 0;
}

/* Implement a C program to create a thread that performs selection sort on an array of integers? */

#include <stdio.h>
#include <pthread.h>

#define MAX_SIZE 100

typedef struct 
{
    int arr[MAX_SIZE];
    int size;
} sort_data_t;

void* selection_sort(void* arg) 
{
    sort_data_t* data = (sort_data_t*)arg;
    int* arr = data->arr;
    int n = data->size;

    for (int i = 0; i < n - 1; i++) 
    {
        int min_idx = i;
        for (int j = i + 1; j < n; j++) 
	{
            if (arr[j] < arr[min_idx]) 
	    {
                min_idx = j;
            }
        }
        int temp = arr[i];
        arr[i] = arr[min_idx];
        arr[min_idx] = temp;
    }

    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    sort_data_t data;

    printf("Enter the number of elements (max %d): ", MAX_SIZE);
    scanf("%d", &data.size);

    if (data.size <= 0 || data.size > MAX_SIZE) 
    {
        printf("Invalid size.\n");
        return 1;
    }

    printf("Enter %d integers:\n", data.size);
    for (int i = 0; i < data.size; i++) 
    {
        scanf("%d", &data.arr[i]);
    }
    if (pthread_create(&thread, NULL, selection_sort, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }
    pthread_join(thread, NULL);
    printf("Sorted array:\n");
    for (int i = 0; i < data.size; i++) 
    {
        printf("%d ", data.arr[i]);
    }
    printf("\n");

    return 0;
}

/* Develop a C program to create a thread that calculates the area of a triangle? */

#include <stdio.h>
#include <pthread.h>

typedef struct 
{
    float base;
    float height;
    float area;
} triangle_data_t;

void* calculate_area(void* arg) 
{
    triangle_data_t* data = (triangle_data_t*)arg;

    data->area = 0.5 * data->base * data->height;

    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    triangle_data_t triangle;
    printf("Enter base of the triangle: ");
    scanf("%f", &triangle.base);

    printf("Enter height of the triangle: ");
    scanf("%f", &triangle.height);
    if (pthread_create(&thread, NULL, calculate_area, &triangle) != 0) 
    {
        perror("pthread_create");
        return 1;
    }
    pthread_join(thread, NULL);
    printf("Area of the triangle = %.2f\n", triangle.area);

    return 0;
}

/* Write a C program to create a thread that generates a random array of integers? */

#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <time.h>

#define MAX_SIZE 100

typedef struct 
{
    int array[MAX_SIZE];
    int size;
} array_data_t;

void* generate_random_array(void* arg) 
{
    array_data_t* data = (array_data_t*)arg;

    srand(time(NULL)); 

    for (int i = 0; i < data->size; i++) 
    {
        data->array[i] = rand() % 100; 
    }

    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    array_data_t data;

    printf("Enter array size (max %d): ", MAX_SIZE);
    scanf("%d", &data.size);

    if (data.size <= 0 || data.size > MAX_SIZE) 
    {
        printf("Invalid array size.\n");
        return 1;
    }
    if (pthread_create(&thread, NULL, generate_random_array, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }
    pthread_join(thread, NULL);
    printf("Generated array:\n");
    for (int i = 0; i < data.size; i++) 
    {
        printf("%d ", data.array[i]);
    }
    printf("\n");

    return 0;
}

/* Develop a C program to create a thread that calculates the greatest common divisor (GCD) of two numbers? */

#include <stdio.h>
#include <pthread.h>

typedef struct 
{
    int a;
    int b;
    int gcd;
} gcd_data_t;

void* compute_gcd(void* arg) 
{
    gcd_data_t* data = (gcd_data_t*)arg;
    int x = data->a;
    int y = data->b;

    while (y != 0) 
    {
        int temp = y;
        y = x % y;
        x = temp;
    }

    data->gcd = x;

    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    gcd_data_t data;

    printf("Enter two integers: ");
    scanf("%d %d", &data.a, &data.b);

    if (data.a <= 0 || data.b <= 0) 
    {
        printf("Please enter positive integers only.\n");
        return 1;
    }

    if (pthread_create(&thread, NULL, compute_gcd, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    printf("GCD of %d and %d is: %d\n", data.a, data.b, data.gcd);

    return 0;
}

/* Implement a C program to create a thread that performs bubble sort on an array of integers? */

#include <stdio.h>
#include <pthread.h>

#define MAX_SIZE 100

typedef struct 
{
    int arr[MAX_SIZE];
    int size;
} sort_data_t;

void* bubble_sort(void* arg) 
{
    sort_data_t* data = (sort_data_t*)arg;
    int* arr = data->arr;
    int n = data->size;

    for (int i = 0; i < n - 1; i++) 
    {
        for (int j = 0; j < n - i - 1; j++) 
	{
            if (arr[j] > arr[j + 1]) 
	    {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
    }

    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    sort_data_t data;

    printf("Enter the number of elements (max %d): ", MAX_SIZE);
    scanf("%d", &data.size);

    if (data.size <= 0 || data.size > MAX_SIZE) 
    {
        printf("Invalid size.\n");
        return 1;
    }

    printf("Enter %d integers:\n", data.size);
    for (int i = 0; i < data.size; i++) 
    {
        scanf("%d", &data.arr[i]);
    }
    if (pthread_create(&thread, NULL, bubble_sort, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }
    pthread_join(thread, NULL);
    printf("Sorted array:\n");
    for (int i = 0; i < data.size; i++) 
    {
        printf("%d ", data.arr[i]);
    }
    printf("\n");

    return 0;
}

/* Write a C program to create a thread that calculates the sum of Fibonacci numbers up to a given limit? */

#include <stdio.h>
#include <pthread.h>

typedef struct 
{
    int limit;
    unsigned long long sum;
} fib_data_t;

void* calculate_fibonacci_sum(void* arg) 
{
    fib_data_t* data = (fib_data_t*)arg;
    int n = data->limit;
    unsigned long long a = 0, b = 1, next;
    data->sum = 0;

    for (int i = 0; i <= n; i++) 
    {
        if (i == 0) 
	{
            next = 0;
        } 
	else if (i == 1) 
	{
            next = 1;
        } 
	else 
	{
            next = a + b;
            a = b;
            b = next;
        }
        data->sum += next;
    }

    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    fib_data_t data;

    printf("Enter the Fibonacci limit (n): ");
    scanf("%d", &data.limit);

    if (data.limit < 0) 
    {
        printf("Invalid input. Limit must be >= 0.\n");
        return 1;
    }
    if (pthread_create(&thread, NULL, calculate_fibonacci_sum, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    printf("Sum of Fibonacci numbers from F(0) to F(%d): %llu\n", data.limit, data.sum);

    return 0;
}

/* Implement a C program to create a thread that calculates the sum of even numbers from 1 to 100? */

#include <stdio.h>
#include <pthread.h>

long long even_sum = 0; 
void* calculate_even_sum(void* arg) 
{
    for (int i = 2; i <= 100; i += 2) 
    {
        even_sum += i;
    }
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;

    if (pthread_create(&thread, NULL, calculate_even_sum, NULL) != 0) 
    {
        perror("pthread_create");
        return 1;
    }
    pthread_join(thread, NULL);
    printf("Sum of even numbers from 1 to 100 is: %lld\n", even_sum);

    return 0;
}

/* Develop a C program to create a thread that calculates the factorial of a given number using iteration? */

#include <stdio.h>
#include <pthread.h>

typedef struct 
{
    int number;
    unsigned long long factorial;
} factorial_data_t;

void* compute_factorial(void* arg) 
{
    factorial_data_t* data = (factorial_data_t*)arg;
    data->factorial = 1;

    for (int i = 1; i <= data->number; i++) 
    {
        data->factorial *= i;
    }

    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    factorial_data_t data;

    printf("Enter a non-negative integer: ");
    scanf("%d", &data.number);

    if (data.number < 0) 
    {
        printf("Factorial is not defined for negative numbers.\n");
        return 1;
    }

    if (pthread_create(&thread, NULL, compute_factorial, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }
    pthread_join(thread, NULL);

    printf("Factorial of %d is: %llu\n", data.number, data.factorial);

    return 0;
}

/* Write a C program to create a thread that checks if a given year is a leap year? */

#include <stdio.h>
#include <pthread.h>

typedef struct 
{
    int year;
} year_data_t;

void* check_leap_year(void* arg) 
{
    year_data_t* data = (year_data_t*)arg;
    int year = data->year;

    if ((year % 4 == 0 && year % 100 != 0) || (year % 400 == 0)) 
    {
        printf("Year %d is a leap year.\n", year);
    } 
    else 
    {
        printf("Year %d is not a leap year.\n", year);
    }

    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    year_data_t data;

    printf("Enter a year: ");
    scanf("%d", &data.year);

    if (pthread_create(&thread, NULL, check_leap_year, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    return 0;
}

/* Implement a C program to create a thread that performs multiplication of two matrices? */

#include <stdio.h>
#include <pthread.h>

#define ROWS 2
#define COLS 2

typedef struct 
{
    int A[ROWS][COLS];
    int B[COLS][COLS];
    int C[ROWS][COLS];
} matrix_data_t;
void* multiply_matrices(void* arg) 
{
    matrix_data_t* data = (matrix_data_t*)arg;

    for (int i = 0; i < ROWS; i++) 
    {
        for (int j = 0; j < COLS; j++) 
	{
            data->C[i][j] = 0;
            for (int k = 0; k < COLS; k++) 
	    {
                data->C[i][j] += data->A[i][k] * data->B[k][j];
            }
        }
    }

    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    matrix_data_t data;

    printf("Enter elements of matrix A (%dx%d):\n", ROWS, COLS);
    for (int i = 0; i < ROWS; i++)
        for (int j = 0; j < COLS; j++)
            scanf("%d", &data.A[i][j]);

    printf("Enter elements of matrix B (%dx%d):\n", COLS, COLS);
    for (int i = 0; i < COLS; i++)
        for (int j = 0; j < COLS; j++)
            scanf("%d", &data.B[i][j]);

    if (pthread_create(&thread, NULL, multiply_matrices, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }
    pthread_join(thread, NULL);

    printf("Resultant matrix C (A x B):\n");
    for (int i = 0; i < ROWS; i++) 
    {
        for (int j = 0; j < COLS; j++) 
	{
            printf("%d ", data.C[i][j]);
        }
        printf("\n");
    }

    return 0;
}

/* Develop a C program to create a thread that calculates the average of numbers from 1 to 100? */

#include <stdio.h>
#include <pthread.h>

typedef struct 
{
    int start;
    int end;
    double average;
} avg_data_t;

void* calculate_average(void* arg) 
{
    avg_data_t* data = (avg_data_t*)arg;
    int sum = 0;

    for (int i = data->start; i <= data->end; i++) 
    {
        sum += i;
    }

    data->average = (double)sum / (data->end - data->start + 1);

    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    avg_data_t data;

    data.start = 1;
    data.end = 100;
    data.average = 0.0;
    if (pthread_create(&thread, NULL, calculate_average, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    printf("Average of numbers from %d to %d is: %.2f\n", data.start, data.end, data.average);

    return 0;
}

/* Implement a C program to create a thread that generates a random string? */

#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <time.h>

#define MAX_LENGTH 100

typedef struct 
{
    int length;
    char result[MAX_LENGTH + 1];
} random_string_data_t;

void* generate_random_string(void* arg) 
{
    random_string_data_t* data = (random_string_data_t*)arg;
    const char charset[] = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";

    for (int i = 0; i < data->length; i++) 
    {
        int key = rand() % (sizeof(charset) - 1);
        data->result[i] = charset[key];
    }
    data->result[data->length] = '\0'; 

    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    random_string_data_t data;

    printf("Enter length of random string (max %d): ", MAX_LENGTH);
    scanf("%d", &data.length);

    if (data.length <= 0 || data.length > MAX_LENGTH) 
    {
        printf("Invalid length.\n");
        return 1;
    }
    srand(time(NULL));

    if (pthread_create(&thread, NULL, generate_random_string, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }


    pthread_join(thread, NULL);

    printf("Random string: %s\n", data.result);

    return 0;
}

/* Write a C program to create a thread that checks if a given number is a perfect square? */

#include <stdio.h>
#include <pthread.h>
#include <math.h>

typedef struct 
{
    int number;
} perfect_square_data_t;

void* check_perfect_square(void* arg) 
{
    perfect_square_data_t* data = (perfect_square_data_t*)arg;
    int num = data->number;

    if (num < 0) 
    {
        printf("%d is not a perfect square (negative number).\n", num);
        pthread_exit(NULL);
    }

    int root = (int)sqrt(num);

    if (root * root == num) 
    {
        printf("%d is a perfect square.\n", num);
    } 
    else 
    {
        printf("%d is not a perfect square.\n", num);
    }

    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    perfect_square_data_t data;

    printf("Enter a number: ");
    scanf("%d", &data.number);

    if (pthread_create(&thread, NULL, check_perfect_square, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    return 0;
}

/* Write a C program to create a thread that calculates the sum of digits of a given number? */

#include <stdio.h>
#include <pthread.h>

typedef struct 
{
    int number;
    int sum;
} sum_digits_data_t;

void* calculate_sum_of_digits(void* arg) 
{
    sum_digits_data_t* data = (sum_digits_data_t*)arg;
    int num = data->number;
    int sum = 0;

    if (num < 0) num = -num;

    while (num > 0) 
    {
        sum += num % 10;
        num /= 10;
    }

    data->sum = sum;
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    sum_digits_data_t data;

    printf("Enter a number: ");
    scanf("%d", &data.number);

    if (pthread_create(&thread, NULL, calculate_sum_of_digits, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    printf("Sum of digits: %d\n", data.sum);

    return 0;
}

/* Implement a C program to create a thread that calculates the factorial of a given number using recursion? */

#include <stdio.h>
#include <pthread.h>

typedef struct 
{
    int number;
    unsigned long long factorial;
} factorial_data_t;

unsigned long long factorial_recursive(int n) 
{
    if (n <= 1)
        return 1;
    else
        return n * factorial_recursive(n - 1);
}

void* calculate_factorial(void* arg) 
{
    factorial_data_t* data = (factorial_data_t*)arg;
    data->factorial = factorial_recursive(data->number);
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    factorial_data_t data;

    printf("Enter a non-negative integer: ");
    scanf("%d", &data.number);

    if (data.number < 0) 
    {
        printf("Factorial is not defined for negative numbers.\n");
        return 1;
    }

    if (pthread_create(&thread, NULL, calculate_factorial, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    printf("Factorial of %d is: %llu\n", data.number, data.factorial);

    return 0;
}

/* Develop a C program to create a thread that finds the maximum element in an array? */

#include <stdio.h>
#include <pthread.h>

#define MAX_SIZE 100

typedef struct 
{
    int arr[MAX_SIZE];
    int size;
    int max;
} max_data_t;

void* find_max(void* arg) 
{
    max_data_t* data = (max_data_t*)arg;
    data->max = data->arr[0];

    for (int i = 1; i < data->size; i++) 
    {
        if (data->arr[i] > data->max) 
	{
            data->max = data->arr[i];
        }
    }

    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    max_data_t data;

    printf("Enter the number of elements (max %d): ", MAX_SIZE);
    scanf("%d", &data.size);

    if (data.size <= 0 || data.size > MAX_SIZE) 
    {
        printf("Invalid size.\n");
        return 1;
    }

    printf("Enter %d elements:\n", data.size);
    for (int i = 0; i < data.size; i++) 
    {
        scanf("%d", &data.arr[i]);
    }

    if (pthread_create(&thread, NULL, find_max, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    printf("Maximum element in the array is: %d\n", data.max);

    return 0;
}

/* Write a C program to create a thread that sorts an array of strings? */

#include <stdio.h>
#include <string.h>
#include <pthread.h>

#define MAX_STRINGS 100
#define MAX_LENGTH 100

typedef struct 
{
    char strings[MAX_STRINGS][MAX_LENGTH];
    int count;
} sort_data_t;

void* sort_strings(void* arg) 
{
    sort_data_t* data = (sort_data_t*)arg;

    for (int i = 0; i < data->count - 1; i++) 
    {
        for (int j = 0; j < data->count - i - 1; j++) 
	{
            if (strcmp(data->strings[j], data->strings[j + 1]) > 0) 
	    {
                char temp[MAX_LENGTH];
                strcpy(temp, data->strings[j]);
                strcpy(data->strings[j], data->strings[j + 1]);
                strcpy(data->strings[j + 1], temp);
            }
        }
    }

    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    sort_data_t data;

    printf("Enter number of strings (max %d): ", MAX_STRINGS);
    scanf("%d", &data.count);
    getchar();

    if (data.count <= 0 || data.count > MAX_STRINGS) 
    {
        printf("Invalid number of strings.\n");
        return 1;
    }

    printf("Enter %d strings:\n", data.count);
    for (int i = 0; i < data.count; i++) 
    {
        fgets(data.strings[i], MAX_LENGTH, stdin);
        data.strings[i][strcspn(data.strings[i], "\n")] = '\0';
    }
    if (pthread_create(&thread, NULL, sort_strings, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }
    pthread_join(thread, NULL);

    printf("Sorted strings:\n");
    for (int i = 0; i < data.count; i++) 
    {
        printf("%s\n", data.strings[i]);
    }

    return 0;
}

/* Implement a C program to create a thread that calculates the square root of a number? */

#include <stdio.h>
#include <pthread.h>
#include <math.h>

typedef struct 
{
    double number;
    double result;
} sqrt_data_t;

void* calculate_sqrt(void* arg) 
{
    sqrt_data_t* data = (sqrt_data_t*)arg;

    if (data->number < 0) 
    {
        printf("Square root of negative number is not real.\n");
        pthread_exit(NULL);
    }

    data->result = sqrt(data->number);
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    sqrt_data_t data;

    printf("Enter a non-negative number: ");
    scanf("%lf", &data.number);

    if (pthread_create(&thread, NULL, calculate_sqrt, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    if (data.number >= 0)
        printf("Square root of %.2f is %.4f\n", data.number, data.result);

    return 0;
}

/* Develop a C program to create a thread that calculates the average of numbers in an array? */

#include <stdio.h>
#include <pthread.h>

#define MAX_SIZE 100

typedef struct 
{
    int arr[MAX_SIZE];
    int size;
    double average;
} average_data_t;

void* calculate_average(void* arg) 
{
    average_data_t* data = (average_data_t*)arg;
    int sum = 0;

    for (int i = 0; i < data->size; i++) 
    {
        sum += data->arr[i];
    }

    data->average = (double)sum / data->size;
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    average_data_t data;

    printf("Enter the number of elements (max %d): ", MAX_SIZE);
    scanf("%d", &data.size);

    if (data.size <= 0 || data.size > MAX_SIZE) 
    {
        printf("Invalid number of elements.\n");
        return 1;
    }

    printf("Enter %d integers:\n", data.size);
    for (int i = 0; i < data.size; i++) 
    {
        scanf("%d", &data.arr[i]);
    }

    if (pthread_create(&thread, NULL, calculate_average, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    printf("Average of the array: %.2f\n", data.average);

    return 0;
}

/* Write a C program to create a thread that generates a random password? */

#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <time.h>

#define MAX_PASSWORD_LENGTH 100

typedef struct 
{
    int length;
    char password[MAX_PASSWORD_LENGTH + 1];
} password_data_t;

void* generate_password(void* arg) 
{
    password_data_t* data = (password_data_t*)arg;
    const char charset[] =
        "abcdefghijklmnopqrstuvwxyz"
        "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
        "0123456789"
        "!@#$%^&*()-_=+[]{}<>?";

    int charset_size = sizeof(charset) - 1;

    for (int i = 0; i < data->length; i++) 
    {
        int key = rand() % charset_size;
        data->password[i] = charset[key];
    }

    data->password[data->length] = '\0';
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    password_data_t data;

    printf("Enter password length (max %d): ", MAX_PASSWORD_LENGTH);
    scanf("%d", &data.length);

    if (data.length <= 0 || data.length > MAX_PASSWORD_LENGTH) 
    {
        printf("Invalid password length.\n");
        return 1;
    }

    srand(time(NULL));

    if (pthread_create(&thread, NULL, generate_password, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    printf("Generated Password: %s\n", data.password);

    return 0;
}

/* Implement a C program to create a thread that checks if a number is even or odd? */

#include <stdio.h>
#include <pthread.h>

typedef struct 
{
    int number;
} number_data_t;

void* check_even_or_odd(void* arg) 
{
    number_data_t* data = (number_data_t*)arg;

    if (data->number % 2 == 0)
        printf("%d is even.\n", data->number);
    else
        printf("%d is odd.\n", data->number);

    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    number_data_t data;

    printf("Enter an integer: ");
    scanf("%d", &data.number);

    if (pthread_create(&thread, NULL, check_even_or_odd, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    return 0;
}

/* Develop a C program to create a thread that calculates the sum of elements in an array? */

#include <stdio.h>
#include <pthread.h>

#define MAX_SIZE 100

typedef struct 
{
    int arr[MAX_SIZE];
    int size;
    int sum;
} sum_data_t;

void* calculate_sum(void* arg) 
{
    sum_data_t* data = (sum_data_t*)arg;
    data->sum = 0;

    for (int i = 0; i < data->size; i++) 
    {
        data->sum += data->arr[i];
    }

    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    sum_data_t data;

    printf("Enter the number of elements (max %d): ", MAX_SIZE);
    scanf("%d", &data.size);

    if (data.size <= 0 || data.size > MAX_SIZE) 
    {
        printf("Invalid number of elements.\n");
        return 1;
    }

    printf("Enter %d integers:\n", data.size);
    for (int i = 0; i < data.size; i++) 
    {
        scanf("%d", &data.arr[i]);
    }

    if (pthread_create(&thread, NULL, calculate_sum, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    printf("Sum of array elements: %d\n", data.sum);

    return 0;
}

/* Write a C program to create a thread that calculates the factorial of numbers from 1 to 10? */

#include <stdio.h>
#include <pthread.h>

void* calculate_factorials(void* arg) 
{
    for (int i = 1; i <= 10; i++) 
    {
        unsigned long long factorial = 1;
        for (int j = 1; j <= i; j++) 
	{
            factorial *= j;
        }
        printf("Factorial of %d is %llu\n", i, factorial);
    }

    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;

    if (pthread_create(&thread, NULL, calculate_factorials, NULL) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    return 0;
}

/* Implement a C program to create a thread that calculates the area of a rectangle */

#include <stdio.h>
#include <pthread.h>

typedef struct 
{
    double length;
    double width;
    double area;
} rectangle_data_t;

void* calculate_area(void* arg) 
{
    rectangle_data_t* data = (rectangle_data_t*)arg;
    data->area = data->length * data->width;
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    rectangle_data_t data;

    printf("Enter length of the rectangle: ");
    scanf("%lf", &data.length);

    printf("Enter width of the rectangle: ");
    scanf("%lf", &data.width);

    if (data.length <= 0 || data.width <= 0) 
    {
        printf("Length and width must be positive numbers.\n");
        return 1;
    }

    if (pthread_create(&thread, NULL, calculate_area, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    printf("Area of the rectangle: %.2f\n", data.area);

    return 0;
}

/* Develop a C program to create a thread that generates random numbers? */

#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <time.h>

#define NUM_COUNT 10

void* generate_random_numbers(void* arg) 
{
    srand(time(NULL) ^ pthread_self());

    printf("Random numbers generated by the thread:\n");
    for (int i = 0; i < NUM_COUNT; i++) 
    {
        int num = rand() % 100; 
        printf("%d ", num);
    }
    printf("\n");

    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;

    if (pthread_create(&thread, NULL, generate_random_numbers, NULL) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    return 0;
}

/* Implement a C program to create a thread that sorts an array of integers?*/

#include <stdio.h>
#include <pthread.h>

#define MAX_SIZE 100

typedef struct 
{
    int arr[MAX_SIZE];
    int size;
} sort_data_t;

void* bubble_sort(void* arg) 
{
    sort_data_t* data = (sort_data_t*)arg;

    for (int i = 0; i < data->size - 1; i++) 
    {
        for (int j = 0; j < data->size - i - 1; j++) 
	{
            if (data->arr[j] > data->arr[j + 1]) 
	    {
                int temp = data->arr[j];
                data->arr[j] = data->arr[j + 1];
                data->arr[j + 1] = temp;
            }
        }
    }

    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    sort_data_t data;

    printf("Enter number of elements (max %d): ", MAX_SIZE);
    scanf("%d", &data.size);

    if (data.size <= 0 || data.size > MAX_SIZE) 
    {
        printf("Invalid number of elements.\n");
        return 1;
    }

    printf("Enter %d integers:\n", data.size);
    for (int i = 0; i < data.size; i++) 
    {
        scanf("%d", &data.arr[i]);
    }

    if (pthread_create(&thread, NULL, bubble_sort, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    printf("Sorted array:\n");
    for (int i = 0; i < data.size; i++) 
    {
        printf("%d ", data.arr[i]);
    }
    printf("\n");

    return 0;
}

/* Write a C program to create a thread that searches for a given number in an array? */

#include <stdio.h>
#include <pthread.h>

#define MAX_SIZE 100

typedef struct 
{
    int arr[MAX_SIZE];
    int size;
    int target;
    int found;
} search_data_t;

void* search_number(void* arg) 
{
    search_data_t* data = (search_data_t*)arg;

    data->found = 0;

    for (int i = 0; i < data->size; i++) 
    {
        if (data->arr[i] == data->target) 
	{
            data->found = 1;
            break;
        }
    }

    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    search_data_t data;

    printf("Enter number of elements (max %d): ", MAX_SIZE);
    scanf("%d", &data.size);

    if (data.size <= 0 || data.size > MAX_SIZE) 
    {
        printf("Invalid number of elements.\n");
        return 1;
    }

    printf("Enter %d integers:\n", data.size);
    for (int i = 0; i < data.size; i++) 
    {
        scanf("%d", &data.arr[i]);
    }

    printf("Enter the number to search: ");
    scanf("%d", &data.target);

    if (pthread_create(&thread, NULL, search_number, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    if (data.found)
        printf("Number %d found in the array.\n", data.target);
    else
        printf("Number %d not found in the array.\n", data.target);

    return 0;
}

/* Develop a C program to create a thread that reverses a given string */

#include <stdio.h>
#include <string.h>
#include <pthread.h>

#define MAX_LEN 100

typedef struct 
{
    char str[MAX_LEN];
} string_data_t;

void* reverse_string(void* arg) 
{
    string_data_t* data = (string_data_t*)arg;
    int len = strlen(data->str);
    for (int i = 0; i < len / 2; i++) 
    {
        char temp = data->str[i];
        data->str[i] = data->str[len - 1 - i];
        data->str[len - 1 - i] = temp;
    }
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    string_data_t data;

    printf("Enter a string (max %d chars): ", MAX_LEN - 1);
    fgets(data.str, MAX_LEN, stdin);

    size_t len = strlen(data.str);
    if (len > 0 && data.str[len - 1] == '\n') 
    {
        data.str[len - 1] = '\0';
    }

    if (pthread_create(&thread, NULL, reverse_string, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    printf("Reversed string: %s\n", data.str);

    return 0;
}

/* Implement a C program to create a thread that reads input from the user? */

#include <stdio.h>
#include <pthread.h>

#define MAX_INPUT 100

typedef struct 
{
    char input[MAX_INPUT];
} input_data_t;

void* read_input(void* arg) 
{
    input_data_t* data = (input_data_t*)arg;
    printf("Enter some input: ");
    if (fgets(data->input, MAX_INPUT, stdin) == NULL) 
    {
        perror("fgets");
    }
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    input_data_t data;

    if (pthread_create(&thread, NULL, read_input, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    printf("You entered: %s", data.input);

    return 0;
}

/* Write a C program to create a thread that performs addition of two matrices? */

#include <stdio.h>
#include <pthread.h>

#define MAX_ROWS 10
#define MAX_COLS 10

typedef struct 
{
    int A[MAX_ROWS][MAX_COLS];
    int B[MAX_ROWS][MAX_COLS];
    int result[MAX_ROWS][MAX_COLS];
    int rows;
    int cols;
} matrix_data_t;

void* add_matrices(void* arg) 
{
    matrix_data_t* data = (matrix_data_t*)arg;
    for (int i = 0; i < data->rows; i++) 
    {
        for (int j = 0; j < data->cols; j++) 
	{
            data->result[i][j] = data->A[i][j] + data->B[i][j];
        }
    }
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    matrix_data_t data;

    printf("Enter number of rows (max %d): ", MAX_ROWS);
    scanf("%d", &data.rows);
    printf("Enter number of columns (max %d): ", MAX_COLS);
    scanf("%d", &data.cols);

    if (data.rows <= 0 || data.rows > MAX_ROWS || data.cols <= 0 || data.cols > MAX_COLS) 
    {
        printf("Invalid matrix dimensions.\n");
        return 1;
    }

    printf("Enter elements of matrix A (%d x %d):\n", data.rows, data.cols);
    for (int i = 0; i < data.rows; i++) 
    {
        for (int j = 0; j < data.cols; j++) 
	{
            scanf("%d", &data.A[i][j]);
        }
    }

    printf("Enter elements of matrix B (%d x %d):\n", data.rows, data.cols);
    for (int i = 0; i < data.rows; i++) 
    {
        for (int j = 0; j < data.cols; j++) 
	{
            scanf("%d", &data.B[i][j]);
        }
    }

    if (pthread_create(&thread, NULL, add_matrices, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    printf("Resultant matrix after addition:\n");
    for (int i = 0; i < data.rows; i++) 
    {
        for (int j = 0; j < data.cols; j++) 
	{
            printf("%d ", data.result[i][j]);
        }
        printf("\n");
    }

    return 0;
}

/* Develop a C program to create a thread that calculates the length of a given string? */

#include <stdio.h>
#include <string.h>
#include <pthread.h>

#define MAX_LEN 100

typedef struct 
{
    char str[MAX_LEN];
    int length;
} string_data_t;

void* calculate_length(void* arg) 
{
    string_data_t* data = (string_data_t*)arg;
    data->length = strlen(data->str);
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    string_data_t data;

    printf("Enter a string (max %d chars): ", MAX_LEN - 1);
    fgets(data.str, MAX_LEN, stdin);

    size_t len = strlen(data.str);
    if (len > 0 && data.str[len - 1] == '\n') 
    {
        data.str[len - 1] = '\0';
    }

    if (pthread_create(&thread, NULL, calculate_length, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    printf("Length of the string: %d\n", data.length);

    return 0;
}

/* Write a C program to create two threads using pthreads library. Each thread should print "Hello, World!" along with its thread ID? */

#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

void* print_hello(void* arg) 
{
    pthread_t tid = pthread_self();
    printf("Hello, World! from thread ID: %lu\n", (unsigned long)tid);
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread1, thread2;

    if (pthread_create(&thread1, NULL, print_hello, NULL) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    if (pthread_create(&thread2, NULL, print_hello, NULL) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);

    return 0;
}

/* Modify the previous program to pass arguments to the threads and print those arguments along with the thread ID? */

#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

typedef struct 
{
    int thread_num;
    char message[50];
} thread_arg_t;

void* print_hello(void* arg) 
{
    pthread_t tid = pthread_self();
    thread_arg_t* t_arg = (thread_arg_t*)arg;

    printf("Thread %d (ID: %lu): %s\n", t_arg->thread_num, (unsigned long)tid, t_arg->message);

    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread1, thread2;

    thread_arg_t arg1 = {1, "Hello from thread 1!"};
    thread_arg_t arg2 = {2, "Hello from thread 2!"};

    if (pthread_create(&thread1, NULL, print_hello, &arg1) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    if (pthread_create(&thread2, NULL, print_hello, &arg2) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);

    return 0;
}

/* Write a C program to demonstrate thread synchronization using mutex locks. Create two threads that increment a shared variable using mutex locks to ensure proper synchronization? */

#include <stdio.h>
#include <pthread.h>

#define NUM_INCREMENTS 1000000

int shared_var = 0;
pthread_mutex_t mutex;

void* increment(void* arg) 
{
    for (int i = 0; i < NUM_INCREMENTS; i++) 
    {
        pthread_mutex_lock(&mutex);
        shared_var++;
        pthread_mutex_unlock(&mutex);
    }
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread1, thread2;

    if (pthread_mutex_init(&mutex, NULL) != 0) 
    {
        perror("Mutex init failed");
        return 1;
    }

    pthread_create(&thread1, NULL, increment, NULL);
    pthread_create(&thread2, NULL, increment, NULL);

    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);

    pthread_mutex_destroy(&mutex);

    printf("Final value of shared_var: %d\n", shared_var);

    return 0;
}


/* Extend the previous program to use semaphore instead of mutex locks for thread synchronization? */

#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>

#define NUM_INCREMENTS 1000000

int shared_var = 0;
sem_t semaphore;

void* increment(void* arg) 
{
    for (int i = 0; i < NUM_INCREMENTS; i++) 
    {
        sem_wait(&semaphore);
        shared_var++;
        sem_post(&semaphore);
    }
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread1, thread2;

    if (sem_init(&semaphore, 0, 1) != 0) 
    {
        perror("Semaphore init failed");
        return 1;
    }

    pthread_create(&thread1, NULL, increment, NULL);
    pthread_create(&thread2, NULL, increment, NULL);

    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);

    sem_destroy(&semaphore);

    printf("Final value of shared_var: %d\n", shared_var);

    return 0;
}

/* Write a C program to implement the producer-consumer problem using pthreads. Create two threads - one for producing items and another for consuming items from a shared buffer? */

#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

#define BUFFER_SIZE 5
#define NUM_ITEMS 20

int buffer[BUFFER_SIZE];
int count = 0;
int in = 0; 
int out = 0; 

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t not_full = PTHREAD_COND_INITIALIZER;
pthread_cond_t not_empty = PTHREAD_COND_INITIALIZER;

void* producer(void* arg) 
{
    for (int i = 1; i <= NUM_ITEMS; i++) 
    {
        pthread_mutex_lock(&mutex);

        while (count == BUFFER_SIZE) 
	{
            pthread_cond_wait(&not_full, &mutex);
        }

        buffer[in] = i;
        in = (in + 1) % BUFFER_SIZE;
        count++;

        printf("Produced: %d (Buffer count: %d)\n", i, count);

        pthread_cond_signal(&not_empty);
        pthread_mutex_unlock(&mutex);

        usleep(100000);
    }
    pthread_exit(NULL);
}

void* consumer(void* arg) 
{
    for (int i = 1; i <= NUM_ITEMS; i++) 
    {
        pthread_mutex_lock(&mutex);

        while (count == 0) 
	{
            pthread_cond_wait(&not_empty, &mutex);
        }

        int item = buffer[out];
        out = (out + 1) % BUFFER_SIZE;
        count--;

        printf("Consumed: %d (Buffer count: %d)\n", item, count);

        pthread_cond_signal(&not_full);
        pthread_mutex_unlock(&mutex);

        usleep(150000);
    }
    pthread_exit(NULL);
}

int main()
{
    pthread_t prod_thread, cons_thread;

    pthread_create(&prod_thread, NULL, producer, NULL);
    pthread_create(&cons_thread, NULL, consumer, NULL);

    pthread_join(prod_thread, NULL);
    pthread_join(cons_thread, NULL);

    printf("Producer-Consumer finished.\n");

    return 0;
}

/* Implement a multithreaded file copy program in C. Create multiple threads to read from one file and write to another file concurrently? */

#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/stat.h>

#define THREAD_COUNT 4
#define CHUNK_SIZE 4096

typedef struct 
{
    int src_fd;
    int dest_fd;
    off_t offset;
    size_t chunk_size;
} thread_arg_t;

void* copy_chunk(void* arg) 
{
    thread_arg_t* t_arg = (thread_arg_t*)arg;
    char* buffer = malloc(t_arg->chunk_size);
    if (!buffer) 
    {
        perror("malloc");
        pthread_exit((void*)1);
    }

    ssize_t bytes_read = pread(t_arg->src_fd, buffer, t_arg->chunk_size, t_arg->offset);
    if (bytes_read < 0) 
    {
        perror("pread");
        free(buffer);
        pthread_exit((void*)1);
    }

    ssize_t bytes_written = pwrite(t_arg->dest_fd, buffer, bytes_read, t_arg->offset);
    if (bytes_written < 0) 
    {
        perror("pwrite");
        free(buffer);
        pthread_exit((void*)1);
    }

    free(buffer);
    pthread_exit(NULL);
}

int main(int argc, char* argv[]) 
{
    if (argc != 3) 
    {
        fprintf(stderr, "Usage: %s <source_file> <dest_file>\n", argv[0]);
        return 1;
    }

    int src_fd = open(argv[1], O_RDONLY);
    if (src_fd < 0) 
    {
        perror("open source");
        return 1;
    }

    int dest_fd = open(argv[2], O_WRONLY | O_CREAT | O_TRUNC, 0644);
    if (dest_fd < 0) 
    {
        perror("open destination");
        close(src_fd);
        return 1;
    }

    struct stat st;
    if (fstat(src_fd, &st) < 0) 
    {
        perror("fstat");
        close(src_fd);
        close(dest_fd);
        return 1;
    }
    off_t file_size = st.st_size;

    if (ftruncate(dest_fd, file_size) < 0) 
    {
        perror("ftruncate");
        close(src_fd);
        close(dest_fd);
        return 1;
    }

    int total_chunks = (file_size + CHUNK_SIZE - 1) / CHUNK_SIZE;

    pthread_t threads[THREAD_COUNT];
    thread_arg_t thread_args[THREAD_COUNT];

    int chunks_per_thread = total_chunks / THREAD_COUNT;
    int remaining_chunks = total_chunks % THREAD_COUNT;

    off_t current_offset = 0;
    int chunk_index = 0;

    for (int i = 0; i < THREAD_COUNT; i++) 
    {
        int chunks_to_copy = chunks_per_thread + (i < remaining_chunks ? 1 : 0);
        for (int j = 0; j < chunks_to_copy; j++, chunk_index++) 
	{
            thread_args[i].src_fd = src_fd;
            thread_args[i].dest_fd = dest_fd;
            thread_args[i].offset = chunk_index * CHUNK_SIZE;

            if (thread_args[i].offset + CHUNK_SIZE > file_size)
                thread_args[i].chunk_size = file_size - thread_args[i].offset;
            else
                thread_args[i].chunk_size = CHUNK_SIZE;

            if (pthread_create(&threads[i], NULL, copy_chunk, &thread_args[i]) != 0) 
	    {
                perror("pthread_create");
                close(src_fd);
                close(dest_fd);
                return 1;
            }
            pthread_join(threads[i], NULL);
        }
    }

    close(src_fd);
    close(dest_fd);

    printf("File copied successfully.\n");
    return 0;
}

/* Write a C program to demonstrate thread cancellation. Create a thread that runs an infinite loop and cancels it after a certain condition is met from the main thread? */

#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

void* worker(void* arg) 
{
    pthread_setcancelstate(PTHREAD_CANCEL_ENABLE, NULL);
    pthread_setcanceltype(PTHREAD_CANCEL_DEFERRED, NULL);

    int count = 0;
    while (1) 
    {
        printf("Worker thread running... count = %d\n", count++);
        sleep(1);

        pthread_testcancel();
    }

    return NULL;
}

int main() 
{
    pthread_t thread;

    if (pthread_create(&thread, NULL, worker, NULL) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    sleep(5);

    printf("Main thread: Canceling worker thread...\n");
    if (pthread_cancel(thread) != 0) 
    {
        perror("pthread_cancel");
        return 1;
    }

    void* res;
    if (pthread_join(thread, &res) != 0) 
    {
        perror("pthread_join");
        return 1;
    }

    if (res == PTHREAD_CANCELED) 
    {
        printf("Worker thread was canceled successfully.\n");
    } else 
    {
        printf("Worker thread terminated normally.\n");
    }

    return 0;
}

/* Write a C program to implement a reader-writer lock using pthreads. Create multiple reader threads and writer threads and ensure proper synchronization using the reader-writer lock? */

#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

#define NUM_READERS 5
#define NUM_WRITERS 2

int shared_data = 0;

typedef struct 
{
    pthread_mutex_t mutex;
    pthread_cond_t readers_proceed;
    pthread_cond_t writer_proceed;
    int readers;
    int writer_active;
    int waiting_writers;
} rwlock_t;

void rwlock_init(rwlock_t* lock) 
{
    pthread_mutex_init(&lock->mutex, NULL);
    pthread_cond_init(&lock->readers_proceed, NULL);
    pthread_cond_init(&lock->writer_proceed, NULL);
    lock->readers = 0;
    lock->writer_active = 0;
    lock->waiting_writers = 0;
}

void rwlock_acquire_readlock(rwlock_t* lock) 
{
    pthread_mutex_lock(&lock->mutex);
    while (lock->writer_active || lock->waiting_writers > 0) 
    {
        pthread_cond_wait(&lock->readers_proceed, &lock->mutex);
    }
    lock->readers++;
    pthread_mutex_unlock(&lock->mutex);
}

void rwlock_release_readlock(rwlock_t* lock) 
{
    pthread_mutex_lock(&lock->mutex);
    lock->readers--;
    if (lock->readers == 0 && lock->waiting_writers > 0) 
    {
        pthread_cond_signal(&lock->writer_proceed);
    }
    pthread_mutex_unlock(&lock->mutex);
}

void rwlock_acquire_writelock(rwlock_t* lock) 
{
    pthread_mutex_lock(&lock->mutex);
    lock->waiting_writers++;
    while (lock->writer_active || lock->readers > 0) 
    {
        pthread_cond_wait(&lock->writer_proceed, &lock->mutex);
    }
    lock->waiting_writers--;
    lock->writer_active = 1;
    pthread_mutex_unlock(&lock->mutex);
}

void rwlock_release_writelock(rwlock_t* lock) 
{
    pthread_mutex_lock(&lock->mutex);
    lock->writer_active = 0;
    if (lock->waiting_writers > 0) 
    {
        pthread_cond_signal(&lock->writer_proceed);
    } 
    else 
    {
        pthread_cond_broadcast(&lock->readers_proceed);
    }
    pthread_mutex_unlock(&lock->mutex);
}

rwlock_t rwlock;

void* reader(void* arg) 
{
    int id = *(int*)arg;
    free(arg);

    while (1) 
    {
        rwlock_acquire_readlock(&rwlock);
        printf("Reader %d: read shared_data = %d\n", id, shared_data);
        rwlock_release_readlock(&rwlock);

        sleep(1);
    }
    return NULL;
}

void* writer(void* arg) 
{
    int id = *(int*)arg;
    free(arg);

    while (1) 
    {
        rwlock_acquire_writelock(&rwlock);
        shared_data++;
        printf("Writer %d: wrote shared_data = %d\n", id, shared_data);
        rwlock_release_writelock(&rwlock);

        sleep(2);
    }
    return NULL;
}

int main() 
{
    pthread_t readers[NUM_READERS], writers[NUM_WRITERS];

    rwlock_init(&rwlock);

    for (int i = 0; i < NUM_READERS; i++) 
    {
        int* id = malloc(sizeof(int));
        *id = i + 1;
        if (pthread_create(&readers[i], NULL, reader, id) != 0) 
	{
            perror("pthread_create reader");
            exit(EXIT_FAILURE);
        }
    }

    for (int i = 0; i < NUM_WRITERS; i++) 
    {
        int* id = malloc(sizeof(int));
        *id = i + 1;
        if (pthread_create(&writers[i], NULL, writer, id) != 0) 
	{
            perror("pthread_create writer");
            exit(EXIT_FAILURE);
        }
    }

    for (int i = 0; i < NUM_READERS; i++) 
    {
        pthread_join(readers[i], NULL);
    }
    for (int i = 0; i < NUM_WRITERS; i++) 
    {
        pthread_join(writers[i], NULL);
    }

    return 0;
}

/* Write a C program to create a thread that prints the even numbers between 1 and 20? */

#include <stdio.h>
#include <pthread.h>

void* print_even_numbers(void* arg) 
{
    printf("Even numbers between 1 and 20:\n");
    for (int i = 1; i <= 20; i++) 
    {
        if (i % 2 == 0) 
	{
            printf("%d ", i);
        }
    }
    printf("\n");
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;

    if (pthread_create(&thread, NULL, print_even_numbers, NULL) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    return 0;
}

/* Develop a C program to create two threads that print odd and even numbers alternately? */

#include <stdio.h>
#include <pthread.h>

#define MAX_NUMBER 20

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
int current = 1;

void* print_odd(void* arg) 
{
    while (current <= MAX_NUMBER) 
    {
        pthread_mutex_lock(&mutex);
        while (current % 2 == 0) 
	{
            pthread_cond_wait(&cond, &mutex);
        }
        if (current <= MAX_NUMBER) 
	{
            printf("Odd thread: %d\n", current++);
        }
        pthread_cond_signal(&cond);
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

void* print_even(void* arg) 
{
    while (current <= MAX_NUMBER) 
    {
        pthread_mutex_lock(&mutex);
        while (current % 2 != 0) 
	{
            pthread_cond_wait(&cond, &mutex);
        }
        if (current <= MAX_NUMBER) 
	{
            printf("Even thread: %d\n", current++);
        }
        pthread_cond_signal(&cond);
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

int main() 
{
    pthread_t odd_thread, even_thread;

    pthread_create(&odd_thread, NULL, print_odd, NULL);
    pthread_create(&even_thread, NULL, print_even, NULL);

    pthread_join(odd_thread, NULL);
    pthread_join(even_thread, NULL);

    return 0;
}

/* Implement a C program to create a thread that calculates the sum of squares of numbers from 1 to 10? */

#include <stdio.h>
#include <pthread.h>

void* sum_of_squares(void* arg) 
{
    int sum = 0;
    for (int i = 1; i <= 10; i++) 
    {
        sum += i * i;
    }
    printf("Sum of squares from 1 to 10 is: %d\n", sum);
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;

    if (pthread_create(&thread, NULL, sum_of_squares, NULL) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    return 0;
}

/* Write a C program to create a thread that calculates the product of numbers from 1 to 5? */

#include <stdio.h>
#include <pthread.h>

void* calculate_product(void* arg) 
{
    int product = 1;
    for (int i = 1; i <= 5; i++) 
    {
        product *= i;
    }
    printf("Product of numbers from 1 to 5 is: %d\n", product);
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;

    if (pthread_create(&thread, NULL, calculate_product, NULL) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    return 0;
}

/* Develop a C program to create a thread that prints the first 10 terms of the Fibonacci sequence? */

#include <stdio.h>
#include <pthread.h>

#define FIB_COUNT 10

void* print_fibonacci(void* arg) 
{
    int fib[FIB_COUNT];
    fib[0] = 0;
    fib[1] = 1;

    for (int i = 2; i < FIB_COUNT; i++) 
    {
        fib[i] = fib[i - 1] + fib[i - 2];
    }

    printf("First %d terms of the Fibonacci sequence:\n", FIB_COUNT);
    for (int i = 0; i < FIB_COUNT; i++) 
    {
        printf("%d ", fib[i]);
    }
    printf("\n");

    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;

    if (pthread_create(&thread, NULL, print_fibonacci, NULL) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    return 0;
}

/* Write a C program to create a thread that finds the maximum element in a given array? */

#include <stdio.h>
#include <pthread.h>

#define ARRAY_SIZE 10

int numbers[ARRAY_SIZE] = {15, 8, 23, 4, 42, 16, 9, 55, 31, 7};
int max_value;

void* find_maximum(void* arg) 
{
    max_value = numbers[0];
    for (int i = 1; i < ARRAY_SIZE; i++) 
    {
        if (numbers[i] > max_value) 
	{
            max_value = numbers[i];
        }
    }
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;

    if (pthread_create(&thread, NULL, find_maximum, NULL) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    printf("Maximum element in the array: %d\n", max_value);

    return 0;
}

/* Implement a C program to create a thread that prints the ASCII values of characters in a given string? */

#include <stdio.h>
#include <pthread.h>
#include <string.h>

void* print_ascii_values(void* arg) 
{
    char* str = (char*)arg;
    printf("ASCII values of characters in the string:\n");
    for (int i = 0; i < strlen(str); i++) 
    {
        printf("Character: '%c' => ASCII: %d\n", str[i], (int)str[i]);
    }
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    char input[100];

    printf("Enter a string: ");
    fgets(input, sizeof(input), stdin);
    input[strcspn(input, "\n")] = '\0';

    if (pthread_create(&thread, NULL, print_ascii_values, (void*)input) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    return 0;
}

/* Develop a C program to create a thread that calculates the sum of all prime numbers up to a given limit? */

#include <stdio.h>
#include <pthread.h>
#include <stdbool.h>

int limit;        
int prime_sum = 0;

bool is_prime(int n) 
{
    if (n <= 1) return false;
    if (n == 2) return true;
    for (int i = 2; i * i <= n; i++) 
    {
        if (n % i == 0)
            return false;
    }
    return true;
}

void* calculate_prime_sum(void* arg) 
{
    for (int i = 2; i <= limit; i++) 
    {
        if (is_prime(i)) 
	{
            prime_sum += i;
        }
    }
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;

    printf("Enter the upper limit to sum prime numbers: ");
    scanf("%d", &limit);

    if (pthread_create(&thread, NULL, calculate_prime_sum, NULL) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    printf("Sum of prime numbers up to %d is: %d\n", limit, prime_sum);

    return 0;
}

/* Write a C program to create a thread that calculates the area of a circle using a given radius? */

#include <stdio.h>
#include <pthread.h>
#include <math.h>

double radius;
double area;

void* calculate_area(void* arg) 
{
    area = M_PI * radius * radius;
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;

    printf("Enter the radius of the circle: ");
    scanf("%lf", &radius);

    if (pthread_create(&thread, NULL, calculate_area, NULL) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    printf("Area of the circle with radius %.2lf is: %.2lf\n", radius, area);

    return 0;
}

/* Develop a C program to create a thread that calculates the average of a given array of floating-point numbers? */

#include <stdio.h>
#include <pthread.h>

typedef struct 
{
    float* array;
    int size;
    float average;
} AvgData;

void* calculate_average(void* arg) 
{
    AvgData* data = (AvgData*)arg;
    float sum = 0.0;

    for (int i = 0; i < data->size; i++) 
    {
        sum += data->array[i];
    }

    data->average = sum / data->size;
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    AvgData data;

    float numbers[] = {12.5, 15.0, 20.75, 10.0, 30.25};
    data.array = numbers;
    data.size = sizeof(numbers) / sizeof(numbers[0]);

    if (pthread_create(&thread, NULL, calculate_average, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    printf("Average of the array is: %.2f\n", data.average);

    return 0;
}

/* Implement a C program to create a thread that prints the factors of a given number? */

#include <stdio.h>
#include <pthread.h>

int number;

void* print_factors(void* arg) 
{
    printf("Factors of %d are: ", number);
    for (int i = 1; i <= number; i++) 
    {
        if (number % i == 0) 
	{
            printf("%d ", i);
        }
    }
    printf("\n");
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;

    printf("Enter a number: ");
    scanf("%d", &number);

    if (pthread_create(&thread, NULL, print_factors, NULL) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);
    return 0;
}

/* Develop a C program to create a thread that prints the English alphabet in uppercase? */

#include <stdio.h>
#include <pthread.h>

void* print_alphabet(void* arg) 
{
    printf("English Alphabet in uppercase:\n");
    for (char ch = 'A'; ch <= 'Z'; ch++) 
    {
        printf("%c ", ch);
    }
    printf("\n");
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;

    if (pthread_create(&thread, NULL, print_alphabet, NULL) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);
    return 0;
}

/* Implement a C program to create a thread that checks if a given number is a perfect square? */

#include <stdio.h>
#include <pthread.h>
#include <math.h>
#include <stdbool.h>

int number;
bool is_perfect_square = false;

void* check_perfect_square(void* arg) 
{
    int n = number;
    int root = (int)sqrt(n);
    if (root * root == n) 
    {
        is_perfect_square = true;
    }
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;

    printf("Enter a number: ");
    scanf("%d", &number);

    if (pthread_create(&thread, NULL, check_perfect_square, NULL) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    if (is_perfect_square) 
    {
        printf("%d is a perfect square.\n", number);
    } else 
    {
        printf("%d is NOT a perfect square.\n", number);
    }

    return 0;
}

/* Implement a C program to create a thread that checks if a given number is divisible by another given number? */

#include <stdio.h>
#include <pthread.h>
#include <stdbool.h>

typedef struct 
{
    int dividend;
    int divisor;
    bool divisible;
} DivData;

void* check_divisible(void* arg) 
{
    DivData* data = (DivData*)arg;
    if (data->divisor == 0) 
    {
        data->divisible = false;
    } 
    else 
    {
        data->divisible = (data->dividend % data->divisor == 0);
    }
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    DivData data;

    printf("Enter dividend: ");
    scanf("%d", &data.dividend);
    printf("Enter divisor: ");
    scanf("%d", &data.divisor);

    if (pthread_create(&thread, NULL, check_divisible, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    if (data.divisor == 0) 
    {
        printf("Division by zero is undefined.\n");
    } 
    else if (data.divisible) 
    {
        printf("%d is divisible by %d.\n", data.dividend, data.divisor);
    } 
    else 
    {
        printf("%d is NOT divisible by %d.\n", data.dividend, data.divisor);
    }

    return 0;
}

/* Develop a C program to create a thread that prints the multiplication table of a given number up to a given limit? */

#include <stdio.h>
#include <pthread.h>

typedef struct 
{
    int number;
    int limit;
} TableData;

void* print_multiplication_table(void* arg) 
{
    TableData* data = (TableData*)arg;
    printf("Multiplication table of %d up to %d:\n", data->number, data->limit);
    for (int i = 1; i <= data->limit; i++) 
    {
        printf("%d x %d = %d\n", data->number, i, data->number * i);
    }
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;
    TableData data;

    printf("Enter the number: ");
    scanf("%d", &data.number);
    printf("Enter the limit: ");
    scanf("%d", &data.limit);

    if (pthread_create(&thread, NULL, print_multiplication_table, &data) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);

    return 0;
}

/*  Develop a C program to create a thread that prints the current system time */

#include <stdio.h>
#include <pthread.h>
#include <time.h>

void* print_current_time(void* arg) 
{
    time_t now;
    time(&now);
    struct tm* local_time = localtime(&now);
    if (local_time == NULL) 
    {
        perror("localtime");
        pthread_exit(NULL);
    }
    printf("Current system time: %02d:%02d:%02d\n",
           local_time->tm_hour, local_time->tm_min, local_time->tm_sec);
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;

    if (pthread_create(&thread, NULL, print_current_time, NULL) != 0) 
    {
        perror("pthread_create");
        return 1;
    }

    pthread_join(thread, NULL);
    return 0;
}

/* Write a C program to create two threads that increment and decrement a shared variable,respectively, using mutex locks? */

#include <stdio.h>
#include <pthread.h>

#define NUM_ITERATIONS 1000000

int shared_var = 0;
pthread_mutex_t lock;

void* increment(void* arg) 
{
    for (int i = 0; i < NUM_ITERATIONS; i++) 
    {
        pthread_mutex_lock(&lock);
        shared_var++;
        pthread_mutex_unlock(&lock);
    }
    pthread_exit(NULL);
}

void* decrement(void* arg) 
{
    for (int i = 0; i < NUM_ITERATIONS; i++) 
    {
        pthread_mutex_lock(&lock);
        shared_var--;
        pthread_mutex_unlock(&lock);
    }
    pthread_exit(NULL);
}

int main() 
{
    pthread_t inc_thread, dec_thread;

    if (pthread_mutex_init(&lock, NULL) != 0) 
    {
        perror("Mutex init failed");
        return 1;
    }

    pthread_create(&inc_thread, NULL, increment, NULL);
    pthread_create(&dec_thread, NULL, decrement, NULL);

    pthread_join(inc_thread, NULL);
    pthread_join(dec_thread, NULL);

    pthread_mutex_destroy(&lock);

    printf("Final value of shared_var: %d\n", shared_var);

    return 0;
}

/* Develop a C program to create a thread that prints numbers from 10 to 1 in descending order using mutex locks? */

#include <stdio.h>
#include <pthread.h>

pthread_mutex_t lock;

void* print_descending(void* arg) 
{
    pthread_mutex_lock(&lock);
    for (int i = 10; i >= 1; i--) 
    {
        printf("%d\n", i);
    }
    pthread_mutex_unlock(&lock);
    pthread_exit(NULL);
}

int main() 
{
    pthread_t thread;

    if (pthread_mutex_init(&lock, NULL) != 0) 
    {
        perror("Mutex init failed");
        return 1;
    }

    if (pthread_create(&thread, NULL, print_descending, NULL) != 0) 
    {
        perror("pthread_create failed");
        return 1;
    }

    pthread_join(thread, NULL);

    pthread_mutex_destroy(&lock);

    return 0;
}
