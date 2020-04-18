Student Name: Patel Tejaskumar Vijaykumar
Student ID: 11802236
Email Address: tejasvpatel711@gmail.com
GitHub Link: https://github.com/Tejaspatel711/Operating-System

Question:

Ques. 11. Reena’s operating system uses an algorithm for deadlock avoidance to manage the allocation of resources say three namely A, B, and C to three processes P0, P1, and P2. Consider the following scenario as reference .user must enter the current state of system as given in this example:
Suppose P0 has 0,0,1 instances, P1 is having 3,2,0 instances and P2 occupies 2,1,1 instances of A, B, C resource respectively. Also, the maximum number of instances required for P0 is 8,4,3 and for p1 is 6,2,0 and finally for P2 there are 3,3,3 instances of resources A, B, C respectively. There are 3 instances of resource A, 2 instances of resource B and 2 instances of resource C available. Write a program to check whether Reena’s operating system is in a safe state or not in the following independent requests for additional resources in the current state: 
1. Request1: P0 requests 0 instances of A and 0 instances of B and 2 instances of C. 
2. Request2: P1 requests for 2 instances of A, 0 instances of B and 0 instances of C. 
All the request must be given by user as input.

Code:

#include <assert.h>
#include <memory.h>
#include <stdbool.h>
#include <stdio.h>
#include <stdlib.h>

#ifdef VERBOSE_ENABLED
#define LOG(...) fprintf(stderr, __VA_ARGS__)
#else
#define LOG(...) \
  do {           \
  } while (false);
#endif

/*
 This is a wrapper that will hold all the information about the current system state
 */
struct system_state {
  int resource_count;
  int process_count;
  int* avail_resource;
  int** allocation_table;
  int** max_table;

} * global_system_state, *global_transient_state;

/* This is a wrapper that holds the request information that is made
 */
struct request {
  int process_id;
  int* resource_requests;

} * request_info;

/* This function free all the dynamically allocated resources.
 */
void free_dynamic_resource() {
  for (int a = 0; a < global_system_state->process_count; a++) {
    free(global_system_state->allocation_table[a]);
    free(global_system_state->max_table[a]);
    free(global_transient_state->max_table[a]);
    free(global_transient_state->allocation_table[a]);
  }
  free(global_system_state->allocation_table);
  free(global_transient_state->allocation_table);
  free(global_system_state->max_table);
  free(global_transient_state->max_table);
  free(global_system_state->avail_resource);
  free(global_transient_state->avail_resource);
  free(global_system_state);
  free(global_transient_state);
}

/**
  This is function asks for the input of the user and allocates and fills the system state.
 * */
void input() {
  LOG("\nStarted input function...");

  global_system_state =
      (struct system_state*)malloc(sizeof(struct system_state));

  global_transient_state =
      (struct system_state*)malloc(sizeof(struct system_state));

  LOG("\nAllocated global_system_state variable...");

  printf("Enter Total Process Count : ");
  scanf("%d", &(global_system_state->process_count));
  global_transient_state->process_count = global_system_state->process_count;
  printf("\nEnter Total Resource Count : ");
  scanf("%d", &(global_system_state->resource_count));
  global_transient_state->resource_count = global_system_state->resource_count;
  LOG("\nRead Process and resource counts...");
  printf(
      "\nEnter Allocated Resource Count as Table (Process in rows, and "
      "Resource in columns) : \n");
  LOG("\nAllocating memory for allocation table...");

  global_system_state->allocation_table =
      (int**)malloc(global_system_state->process_count * sizeof(int*));

  global_transient_state->allocation_table =
      (int**)malloc(global_transient_state->process_count * sizeof(int*));

  for (int a = 0; a < global_system_state->process_count; a++) {
    global_system_state->allocation_table[a] =
        (int*)malloc(global_system_state->resource_count * sizeof(int));
    global_transient_state->allocation_table[a] =
        (int*)malloc(global_transient_state->resource_count * sizeof(int));
  }

  LOG("\nAllocated allocation table memory now reading...");
  // Read the allocation table
  for (int a = 0; a < global_system_state->process_count; a++)
    for (int b = 0; b < global_system_state->resource_count; b++) {
      scanf("%d", &(global_system_state->allocation_table[a][b]));
      global_transient_state->allocation_table[a][b] =
          global_system_state->allocation_table[a][b];
    }
  LOG("\nMax Table Allocating memory...");
  // Allocate memory for Max resource table

  global_system_state->max_table =
      (int**)malloc(global_system_state->process_count * sizeof(int*));
  global_transient_state->max_table =
      (int**)malloc(global_transient_state->process_count * sizeof(int*));

  for (int a = 0; a < global_system_state->process_count; a++) {
    global_system_state->max_table[a] =
        (int*)malloc(global_system_state->resource_count * sizeof(int));
    global_transient_state->max_table[a] =
        (int*)malloc(global_transient_state->resource_count * sizeof(int));
  }

  LOG("\nAllocated max table memory now reading...");
  printf(
      "\nEnter Maximum Resource Count Limit as Table (Process in rows, and "
      "Resource limit in columns) : \n");

  for (int a = 0; a < global_system_state->process_count; a++)
    for (int b = 0; b < global_system_state->resource_count; b++) {
      scanf("%d", &(global_system_state->max_table[a][b]));
      global_transient_state->max_table[a][b] =
          global_system_state->max_table[a][b];
    }

  LOG("\nAllocating memory to avail_resources...");
  // Allocate memory for avail resource vector and list;
  global_system_state->avail_resource =
      (int*)malloc(sizeof(int) * global_system_state->resource_count);
  global_transient_state->avail_resource =
      (int*)malloc(sizeof(int) * global_transient_state->resource_count);

  LOG("\nReading values to available_resources...");

  printf(
      "\nEnter available resource count in the same order as above for each "
      "resource : \n");
  for (int a = 0; a < global_system_state->resource_count; a++) {
    scanf("%d", &(global_system_state->avail_resource[a]));
    global_transient_state->avail_resource[a] =
        global_system_state->avail_resource[a];
  }
};

/**
 This is a function responsible for actually finding the stable state if found
it sets the solution_state with the state else sets it to NULL
 * */

/*
These are some useful math functions for easing the solving task
a : available;
b : allocated;
c : required;
*/
bool vec_math_is_allocatable(int* a, int* b, int* c, int len) {
  for (int iter = 0; iter < len; iter++)
    if (a[iter] + b[iter] < c[iter]) return false;
  return true;
}
/*
These are some useful math functions for easing the solving task
a : available;
b : allocated;
c : required;
*/
void vec_math_allocate_and_free(int* a, int* b, int* c, int len) {
  assert(vec_math_is_allocatable(a, b, c, len));
  for (int iter = 0; iter < len; iter++) {
    a[iter] += b[iter];
    b[iter] = 0;
  }
}

/*
  a : allocated
  b : requested
  c : max_limit
 */

bool vec_math_should_grant(int* a, int* b, int* c, int len) {
  for (int t = 0; t < len; t++)
    if (a[t] + b[t] > c[t]) return false;
  return true;
}

/*
  This function restores the state to old state onces a request has been
 processed
 */

void restore() {
  LOG("\nRestoring Available Resource State to Global State...");

  for (int a = 0; a < global_system_state->resource_count; a++)
    global_transient_state->avail_resource[a] =
        global_system_state->avail_resource[a];

  LOG("\nResources Allocation Table Restored...");

  for (int a = 0; a < global_system_state->process_count; a++)
    for (int b = 0; b < global_system_state->resource_count; b++)
      global_transient_state->allocation_table[a][b] =
          global_system_state->allocation_table[a][b];
}
/*
 This function solves the state after the request has been granted and returns
 true if system is in stable state or else false if deadlock is encountered
 */
bool solve() {
  LOG("\nStarted solving the system...");
  int non_executed = global_system_state->process_count;
  bool dead_lock = false;
  bool has_completed[global_system_state->process_count];

  for (int a = 0; a < global_system_state->process_count; a++)
    has_completed[a] = false;

  while (non_executed) {
    dead_lock = true;
    for (int a = 0; a < global_system_state->process_count; a++) {
      if (has_completed[a]) continue;
      LOG("\nChecking if P%d can be allocated for execution...", a);
      bool res =
          vec_math_is_allocatable(global_transient_state->avail_resource,
                                  global_transient_state->allocation_table[a],
                                  global_transient_state->max_table[a],
                                  global_transient_state->resource_count);
      if (res) {
        has_completed[a] = true;
        non_executed--;
        dead_lock = false;
        LOG("\nAllocating and releasing resources for P%d", a);

        vec_math_allocate_and_free(global_transient_state->avail_resource,
                                   global_transient_state->allocation_table[a],
                                   global_transient_state->max_table[a],
                                   global_transient_state->resource_count);
      } else {
        LOG("\nCannot satisfy needs for P%d. Skipping...", a);
      }
    }
    if (dead_lock) {
      LOG("\nEncountered a deadlock...");
      return false;
    }
  }
  return true;
};

/*
  This asks the request count from user.
 */

void ask_request_count(int* target) {
  printf("\nHow many number of requests will arrive : ");
  scanf("%d", target);
}
/*
  This asks the actual request and solves the state and prints if it is safe
 state or not after request has been granted
 */
void ask_requests(int n) {
  for (int t = 0; t < n; t++) {
    printf("\nRequest %d : ", t + 1);
    int p_c;
    printf("\nEnter Process ID which request resources : ");
    scanf("%d", &p_c);
    p_c--;

    if (p_c > global_system_state->process_count) {
      printf("\nOpps !! No such PID found");
      t--;
      continue;
    }

    printf("\nEnter %d space separated integers each for each resource type : ",
           global_system_state->resource_count);
    request_info = (struct request*)malloc(sizeof(struct request));
    request_info->process_id = p_c;
    request_info->resource_requests =
        (int*)malloc(sizeof(int) * global_system_state->resource_count);

    for (int a = 0; a < global_system_state->resource_count; a++)
      scanf("%d", &(request_info->resource_requests[a]));

    if (vec_math_should_grant(global_system_state->allocation_table[p_c],
                              request_info->resource_requests,
                              global_system_state->max_table[p_c],
                              global_system_state->resource_count)) {
      bool flag = true;
      for (int a = 0; a < global_system_state->resource_count; a++)
        if (global_transient_state->avail_resource[a] <
            request_info->resource_requests[a]) {
          flag = false;
          break;
        }

      if (flag)
        printf("\nGranting the Resources. We have Enough resource to grant.");
      else
        printf(
            "\nPartially Granted Request. Limit increased in the allocation "
            "table");

      for (int a = 0; a < global_system_state->resource_count; a++) {
        if (flag)
          global_transient_state->avail_resource[a] -=
              request_info->resource_requests[a];
        global_transient_state->allocation_table[p_c][a] +=
            request_info->resource_requests[a];
      }

      if (!solve())
        printf(
            "\nDEADLOCK : After Request was Granted the system went into "
            "DEADLOCK");
      else
        printf(
            "\nSystem has a Stable State even after the resource requested was "
            "granted");
    }

    else {
      printf(
          "\nRequest for the resources denied... (Requested more than limit)");
    }

    free(request_info->resource_requests);
    free(request_info);
    restore();
    printf("\n");
  }
}

/*
  This is main driver program.
*/
int main() {
  input();
  if (solve())
    printf("\nInitially system is in Safe State");
  else {
    printf("\nPanic Deadlock initially.");
    return 0;
  }
  restore();
  int n;
  ask_request_count(&n);
  ask_requests(n);
  free_dynamic_resource();
  return 0;
}


Description:
The question which is given to solve is based on the principle of deadlock avoidance, deadlock detection and Banker’s algorithm. Banker’s Algorithm is a deadlock avoidance algorithm. It maintains a set of data using which it decides whether to entertain the request of any process or not. It follows the safety algorithm to check whether the system is in a safe state or not. The banker’s algorithm is a resource allocation and deadlock avoidance algorithm that tests for safety by simulating the allocation for predetermined maximum possible amounts of all resources, then makes an “s-state” check to test for possible activities, before deciding whether allocation should be allowed to continue. Banker’s algorithm is named so because it is used in banking system to check whether loan can be sanctioned to a person or not. Suppose there are n number of account holders in a bank and the total sum of their money is S. If a person applies for a loan then the bank first subtracts the loan amount from the total money that bank has and if the remaining amount is greater than S then only the loan is sanctioned. It is done because if all the account holders come to withdraw their money then the bank can easily do it.

Algorithm:
Banker’s Algorithm:
Banker’s Algorithm is a resource allocation and deadlock avoidance algorithm. This algorithm test for safety simulating the allocation for predetermined maximum possible amounts of all resources, then makes an “s-state” check to test for possible activities, before deciding whether allocation should be allowed to continue.In simple terms, it checks if allocation of any resource will lead to deadlock or not, OR is it safe to allocate a resource to a process and if not then resource is not allocated to that process. Determining a safe sequence(even if there is only 1) will assure that system will not go into deadlock.Banker’s algorithm is generally used to find if a safe sequence exist or not. But here we will determine the total number of safe sequences and print all safe sequences.



Algorithm:

1) Let Work and Finish be vectors of length ‘m’ and ‘n’ respectively.
Initialize: Work = Available
Finish[i] = false; for i=1, 2, 3, 4….n
2) Find an i such that both
a) Finish[i] = false
b) Needi <= Work
if no such i exists goto step (4)
3) Work = Work + Allocation[i]
Finish[i] = true
goto step (2)
4) if Finish [i] = true for all i
then the system is in a safe state

Time Complexity:
Time complexity = O(n*n*m), where n = number of processes and m = number of resources.

Implementation of Code:



Boundary Conditions:

If we permit Request1 then the state will be:

AVAILABLE		X=3, Y=2, Z=0
	  MAX	  ALLOCATION	 NEED
	  X Y	Z	  X	Y	Z	    X	Y	Z
P0	8	4	3	  0	0	3	    8	4	0
P1	6	2	0	  3	2	0	    3	0	0
P2	3	3	3	  2	1	1 	  1	2	2

Now, with the current availability, we can service the need of P1. The state would become:

AVAILABLE		X=6, Y=4, Z=0
	  MAX 	ALLOCATION	 NEED
	  X Y	Z 	X	Y	Z	    X	Y	Z
P0	8	4	3	  0	0	3	    8	4	0
P1	6	2	0	  3	2	0	    0	0	0
P2	3	3	3	  2	1	1	    1	2	2

With the resulting availability, it would not be possible to service the need of either P0 or P2, owing to lack of Z resource.
Therefore, the system would be in a deadlock.
⇒ We cannot permit REQ1.
Test cases:
If we permit to Request2, then

AVAILABLE		X=1, Y=2, Z=2
	  MAX 	ALLOCATION	 NEED
	  X Y	Z 	X	Y	Z 	  X	Y	Z
P0	8	4	3 	0	0	1 	  8	4	2
P1	6	2	0	  5	2	0 	  1	0	0
P2	3	3	3	  2	1	1 	  1	2	2

With this availability, we service P1 (P2 can also be serviced). So, the state is :

AVAILABLE		X=6, Y=4, Z=2
	   MAX ALLOCATION	NEED
	  X	Y	Z	  X	Y	Z 	X	Y	Z
P0	8	4	3	  0	0	1 	8	4	2
P1	6	2	0	  5	2	0	  0	0	0
P2	3	3	3	  2	1	1	  1	2	2


With the current availability, we service P2. The state becomes:

AVAILABLE		X=8, Y=5, Z=3
	   MAX	ALLOCATION	NEED
	  X Y	Z	  X	Y	Z	  X	Y	Z
P0	8	4	3	  0	0	1	  8	4	2
P1	6	2	0	  5	2	0	  0	0	0
P2	3	3	3	  2	1	1	  0	0	0

Finally, we service P0. The state now becomes:

AVAILABLE		X=8, Y=5, Z=4
	  MAX 	ALLOCATION	NEED
	  X Y	Z	  X	Y	Z	  X	Y	Z
P0	8	4	3	  0	0	1	  0	0	0
P1	6	2	0	  5	2	0 	0	0	0
P2	3	3	3	  2	1	1 	0	0	0

The state so obtained is a safe state. ⇒ REQ2 can be permitted.
So, only REQ2 can be permitted.
GitHub Link:	 https://github.com/Tejaspatel711/Operating-System
