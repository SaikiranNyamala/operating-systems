# operating-systems
3.	Write a multithreaded program that calculates various statistical values for a list of numbers. This program will be passed a series of numbers on the command line and will then create three separate worker threads. One thread will determine the average of the numbers, the second will determine the maximum value, and the third will determine the minimum value. For example, suppose your program is passed the integers  
90 81 78 95 79 72 85 
The program will report 
The average value is 82 
The minimum value is 72 
The maximum value is 95  
The variables representing the average, minimum, and maximum values will be stored globally. The worker threads will set these values, and the parent thread will output the values once the workers have exited.  

code:
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <errno.h>
#define handle_error_en(en, msg) \
        do { errno = en; perror(msg); exit(EXIT_FAILURE); } while (0)
volatile int running_threads = 0;

pthread_t thread[3]; 

int E;

struct Results{ 
	
	int min;
	int max;
	int average;

}Results;

void *findMin(void *array_ptr){
	
	int i; 
	
	int *elements = (int*)array_ptr; 

	Results.min = elements[0]; 

	for(i = 0; i < E; i++){	

		if(elements[i] < Results.min){	

			Results.min = elements[i];	
	
		}
	
	}

	running_threads -= 1;

return NULL;

}
void *findMax(void *array_ptr){
	
	int i;

	int *elements = (int*)array_ptr; 
	
	for(i = 0; i < E; i++){

		if(elements[i] > Results.max){

			Results.max = elements[i];

		}
	}

	running_threads -= 1;	

return NULL;

}

void *findAverage(void *array_ptr){
	
	int i;	 

	int *elements = (int*)array_ptr; 

	for(i = 0; i < E; i++){		

		Results.average += elements[i];	

	}

	Results.average = Results.average/E;	

	running_threads -= 1;	

return NULL;

}

int getArrayInput(int n, int *array_ptr){
		
		int input;

		int E = 0;

    	printf("Creating Dynamic Array...\enter a negative numbern-\n");
		printf("Enter a positive value:\n");
		for(;;){ 
   

			if (scanf("%d",&input) != 1){

				printf("\nOops that wasn't an Integer\nlets try filling the array again\nRemember INTEGERS only!\n");

				exit(EXIT_FAILURE);
			
			}

    		if (input >= 0){ 

       		 	if (E == n){

          	  	  n += 1; 
            		
          		  array_ptr = realloc(array_ptr, n * sizeof(int));
				            
       			 }

        		array_ptr[E++] = input;
    
    		} else {
        
       		 printf("\nNumber of Integers: %d\n", E);
       
       		 break;

   				 }

			}

	return E;
	
		}

void joinThreads(int numberOfThreads){
	
	int i;

	int s; 

	while(numberOfThreads >= 0){	

		s = pthread_join(thread[numberOfThreads], NULL);

		 
		 if (s != 0){
		 	
			handle_error_en(s, "pthread_create");
		 
		 }

		 numberOfThreads--;

	}
	
}
void createThreads(int *array_ptr){
	
	int s;  
 
 	s = pthread_create(&thread[0], NULL, findMin, (void *)array_ptr);

	 if (s != 0){

			handle_error_en(s, "pthread_create");
		 
		 }
		 	running_threads += 1;

	 
	 s = pthread_create(&thread[1], NULL, findMax, (void *)array_ptr);

		 if (s != 0){
        
            handle_error_en(s, "pthread_create");
       	
       	 }
        	 running_threads += 1;

	 
	 s = pthread_create(&thread[2], NULL, findAverage, (void *)array_ptr);
	 		 
		 if (s != 0){

           handle_error_en(s, "pthread_create");
       	
       	 }
			
			running_threads += 1;

}
int main(){

	int n = 1; 

	int *array_ptr = malloc(n * sizeof(int));
		
		 
		 E = getArrayInput(n, array_ptr);
		
		 createThreads(array_ptr);
		
	    	while(running_threads>0){
	
				sleep(1);

			}

		joinThreads(2);	
		printf("\nThe average is %d\nThe maximum is %d\nThe minimum is %d\n",Results.average, Results.max, Results.min);

	return(0);

}












//
30)Design a program using concepts of inter-process communication ordinary pipes in which one process sends a string message to a second process, and the second process reverses the case of each character in the message and sends it back to the first process. For example, if the first process sends the message Hi There, the second process will return hI tHERE. This will require using two pipes, one for sending the original message from the first to the second process and the other for sending the 
modified message from the second to the first process. You can write this program using either UNIX or Windows pipes.  

code:
#include <sys/types.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <stdlib.h>
 
#define BUFFER_SIZE 25
#define READ_END 0
#define WRITE_END 1
#define READ_END2 0
#define WRITE_END2 1
 
int main(void)
{
  int ch = 0;
  int ch2 = 0;
  char write_msg[BUFFER_SIZE] = "Greetings";
  char write_msg2[BUFFER_SIZE];
  char read_msg[BUFFER_SIZE];
  int fd[2];
  int fd2[2];
  pid_t pid;
  if (pipe(fd) == -1) {
    fprintf(stderr, "Pipe failed");
    return 1;
  }
  if (pipe(fd2) == -1) {
    fprintf(stderr, "Pipe failed");
    return 1;
  }
  pid = fork();
 
  if (pid < 0) {
    fprintf(stderr, "Fork failed");
    return 1;
  }
 
  if (pid>0){
    close(fd[READ_END]);
    close(fd2[WRITE_END2]);
     
    while (write_msg[ch] != 0)
    {
      write(fd[WRITE_END], &write_msg[ch], strlen(write_msg)+1);
      ch++;
    }
    close(fd[WRITE_END]);
    while (read_msg[ch] != 0)
    {
      read(fd2[READ_END2], &read_msg[ch2], 1);
      printf("%c", read_msg[ch2]);
      if (ch2%2 == 0) {
    ch2++;
      }
    }
  }
 
  else {
    close(fd[WRITE_END]);
    close(fd2[READ_END2]);
    ch = 0;
    ch2 =0;
    while (read_msg[ch] != 0)
    {
      read(fd[READ_END], &read_msg[ch], 1);
      write_msg2[ch] = toupper(&read_msg[ch]);
      printf("%c", read_msg[ch]);
      if (ch%2 == 0) {
    ch++;
      }
    }
    while (write_msg2[ch2] != 0)
    {
      write(fd2[WRITE_END2], &write_msg2[ch2], strlen(write_msg2)+1);
      ch2++;
    }
 
    close(fd2[READ_END]);
  }
  return 0;
}









