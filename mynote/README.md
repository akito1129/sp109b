# 鄒耀文的系統程式筆記

## Git指令

* master改成main

## Markdown

* [Markdown是什麼?](http://programmermedia.org/root/%E9%99%B3%E9%8D%BE%E8%AA%A0/%E6%8A%80%E8%83%BD/markdown.md)

## enum(列舉)

* 預設從'0'開始

## if

* 用compiler.c,參考while的寫法後修改

##

    // if (E) STMT (else STMT)?
    void IF() {
      int ifBegin = nextLabel();
      int ifMid = nextLabel();
      int ifEnd = nextLabel();
      emit("(L%d)\n", ifBegin);
      skip("if");
      skip("(");
      int e = E();
      emit("if not t%d goto L%d\n", e, ifMid);  //未滿足上一個式子，將跳到下一個條件式
      skip(")");
      STMT();
      emit("goto L%d\n", ifEnd);  //符合條件，執行完，離開if
      emit("(L%d)\n", ifMid);  //下一個條件式
      if (isNext("else")) {
        skip("else");
        STMT();
        emit("(L%d)\n", ifEnd);  //離開if
      }
    }
* elif.c

##

    if(a>5){
        t=1;
    }
    else if(a>3){
        t=2;
    }
    else {
        t=3;
    }

* Run

##

    cxz1d@MSI MINGW64 ~/OneDrive/桌面/nohano1l/sp/03-compiler/03b-compiler2 (master)
    $ ./compiler 'test/elif.c'
    if(a>5){
        t=2;
    }
    else {
        t=3;
    }

    ============ parse =============

    (L0)
    t0 = a
    t1 = 5
    t2 = t0 > t1
    if not t2 goto L1
    t3 = 1
    t = t3
    goto L2
    (L1)
    (L3)
    t4 = a
    t5 = 3
    t6 = t4 > t5
    if not t6 goto L4
    t7 = 2
    t = t7
    goto L5
    (L4)
    t8 = 3
    t = t8
    (L5)
    (L2)    

## 何為指標

![README](./1.png)

## 安裝 multipass

* [multipass](https://multipass.run/?_ga=2.30218559.1492289971.1615984144-554578233.1615984144)
* 不行就用其他linux

## 安裝 ubuntu

* [ubuntu](https://www.kjnotes.com/linux/29)

## Linux 用戶和用戶組管理

* [Linux 用戶和用戶組管理](https://www.runoob.com/linux/linux-user-manage.html)

## Multipass虛擬機

* [Multipass虛擬機](https://codingnote.cc/zh-tw/p/287792/)

## 虛擬機、模擬器的區別

    虛擬機:有模擬整套CPU的指令集
    模擬器:模擬電腦外部的行為

##  執行堆疊機 

![README](./2.png)

## gcc編譯流程
![README](./3.jpg)
![README](./4.jpg)
![README](./5.jpg)

## 題外話 : 權利區分

    著作權(版權):作者終生+50年,須具有獨創性
    專利:20年,須具有首創性(新穎、進步、可實行)
    商標:10年(可續展),須具有識別性

## 記憶體配置
![README](./6.png)

## 作業系統
![README](./7.png)

## Process
![README](./8.jpg)
![README](./9.jpg)
![README](./10.jpg)

## 死結 deadlock

* 兩個以上的運算單元，雙方都在等待對方停止執行，以取得系統資源，但是沒有一方提前退出時，就稱為死結。
![README](./11.jpg)
* P1、P2兩個process都需要資源才能繼續執行。P1擁有資源R2、還需要額外資源R1才能執行；P2擁有資源R1、還需要額外資源R2才能執行，兩邊都在互相等待而沒有任何一個可執行。
##
	#include <pthread.h>
	#include <stdio.h>
	#include <unistd.h>

	pthread_mutex_t x;
	pthread_mutex_t y;

	void *A(); 
	void *B(); 

	int main(int argc, char *argv[])
	{
   	 pthread_t threadA, threadB;
   	 pthread_attr_t attr;

   	 pthread_attr_init(&attr);
   	 pthread_mutex_init(&x, NULL);
   	 pthread_mutex_init(&y, NULL);

   	 pthread_create(&threadA, &attr, A, NULL);
   	 pthread_create(&threadB, &attr, B, NULL);
		
   	 pthread_join(threadA, NULL);
   	 pthread_join(threadB, NULL);

   	 pthread_mutex_destroy(&x);
   	 pthread_mutex_destroy(&y);
	}

	void *A() 
	{
   	 pthread_mutex_lock(&x);
  	  printf("A lock x\n");
	
   	 sleep(1);
   	 pthread_mutex_lock(&y);
   	 printf("A lock y\n");

   	 pthread_mutex_unlock(&y); 
  	  pthread_mutex_unlock(&x); 

   	 printf("finished A\n");

   	 pthread_exit(0);
	}

	void *B()
	{
    
   	 pthread_mutex_lock(&y);
  	  printf("B lock y\n");
   	 sleep(1);
  	  pthread_mutex_lock(&x);
  	  printf("B lock x\n");
  	  pthread_mutex_unlock(&x);
   	 pthread_mutex_unlock(&y);

   	 pthread_exit(0);
	}
![README](./deadlock.png)

## race.c
	#include <stdio.h>
	#include <pthread.h>

	#define LOOPS 100000000
	int counter = 0;

	void *inc()
	{
 	 for (int i=0; i<LOOPS; i++) {
    counter = counter + 1;
  	}
 	 return NULL;
	}

	void *dec()
	{
  	for (int i=0; i<LOOPS; i++) {
   	 counter = counter - 1;
  	}
 	 return NULL;
	}


	int main() 
	{
  	pthread_t thread1, thread2;

 	 pthread_create(&thread1, NULL, inc, NULL);
 	 pthread_create(&thread2, NULL, dec, NULL);

 	 pthread_join( thread1, NULL);
  	pthread_join( thread2, NULL);
  	printf("counter=%d\n", counter);
	}
![README](./race.png)
![README](./raced.png)
## norace.c
	#include <stdio.h>
	#include <pthread.h>

	pthread_mutex_t mutex1 = PTHREAD_MUTEX_INITIALIZER;
	#define LOOPS 100000
	int counter = 0;

	void *inc()
	{
 	 for (int i=0; i<LOOPS; i++) {
 	   pthread_mutex_lock( &mutex1 );
 	   counter = counter + 1;
 	   pthread_mutex_unlock( &mutex1 );
 	 }
	  return NULL;
	}

	void *dec()
	{
  	for (int i=0; i<LOOPS; i++) {
  	  pthread_mutex_lock( &mutex1 );
 	   counter = counter - 1;
 	   pthread_mutex_unlock( &mutex1 );
 	 }
	  return NULL;
	}


	int main() 
	{
		pthread_t thread1, thread2;
	
		pthread_create(&thread1, NULL, inc, NULL);
 	 pthread_create(&thread2, NULL, dec, NULL);

 	 pthread_join( thread1, NULL);
 	 pthread_join( thread2, NULL);
 	 printf("counter=%d\n", counter);
	}
![README](./noraced.png)
## 實體位址

* [實體位址](https://zh.wikipedia.org/wiki/%E7%89%A9%E7%90%86%E5%9C%B0%E5%9D%80)

## 邏輯位址

* [邏輯位址](https://zh.wikipedia.org/wiki/%E9%80%BB%E8%BE%91%E5%9C%B0%E5%9D%80)

## 哲學家問題
	#include <stdio.h>
	#include <unistd.h>
	#include <pthread.h>
	#include <semaphore.h>
	typedef	enum { False=0, True=1 } bool ;

	#define N 5
	#define P 3

	sem_t Room;
	sem_t Fork[P];
	bool Switch ;

	void *tphilosopher(void *ptr) {
		int i, k = *((int *) ptr);
		for(i = 1; i <= N; i++) {
			printf("%*cThink %d %d\n", k*4, ' ', k, i);
			if(Switch) {
				sem_wait(&Room) ;
			}
			sem_wait(&Fork[k]) ;
			sem_wait(&Fork[(k+1) % P]) ;
			printf("%*cEat %d %d\n", k*4, ' ', k, i);
			sem_post(&Fork[k]) ;
			sem_post(&Fork[(k+1) % P]) ;
			if(Switch) {
				sem_post(&Room) ;
				sleep(1);
			}
		}
		pthread_exit(0);
	}

	int main(int argc, char * argv[]) {
		int i, targ[P];
		pthread_t thread[P];
		sem_init(&Room, 0, P-1); 
		Switch = (argc > 1);
		printf("Switch=%s\n",(Switch?"true":"false"));
		for(i=0;i<P;i++) {
			sem_init(&Fork[i], 0, 1);
		}
		for(i=0;i<P;i++) {
			targ[i] = i;
			pthread_create(&thread[i], NULL, &tphilosopher,(void *) &targ[i]);
		}
		for(i=0;i<P;i++) {
			pthread_join(thread[i], NULL);
		}
		for(i=0;i<P;i++) {
			sem_destroy(&Fork[i]);
		}
		sem_destroy(&Room);
		return 0;
	}
![README](./12.png)

## fork1.c
![README](./fork1ed.png)

## fork2.c
![README](./fork2ed.png)

## fork3.c
![README](./fork3ed.png)

## twochildfork2.c
![README](./twochildfork2ed.png)

## execvp1.c
![README](./execvp1ed.png)

## stderr1.c
![README](./stderr1ed.png)
## stderr2.c
![README](./stderr2ed.png)
## blocking1.c 
![README](./blocking1ed.png)
## nonblocking1.c
![README](./nonblocking1ed.png)
## nonblocking2.c
![README](./nonblocking2ed.png)
## time.c
![README](./timed.png)
## pipe
![README](./pipe.png)
## fifo
* install screen
##
	sudo apt-get install screen
* 指令
##
	screen : 執行
	exit : 離開
	ctrl + a + c : 新增視窗
	ctrl + a + n : 變換視窗
## chat
![README](./chat.png)

## fifo
	#include <stdio.h>
	#include <string.h>
	#include <fcntl.h>
	#include <sys/stat.h>
	#include <sys/types.h>
	#include <unistd.h>

	#define SMAX 80

	int main(int argc, char *argv[]) {
    	int fd;
    	char *fifo0 = "/tmp/user0";
    	char *fifo1 = "/tmp/user1";
    	mkfifo(fifo0, 0666); 
    	mkfifo(fifo1, 0666);

    	char *me, *you;
		if (strcmp(argv[1], "0")) { // me:0 => you:1
			me = fifo0;
        	you = fifo1;
    	} else { // me:1 => you:0
        	me = fifo1;
       		you = fifo0;
    	}

    	char msg[SMAX];
    	if (fork() == 0) { 
        	fd = open(you, O_RDONLY); 
        	while (1) {
            	int n = read(fd, msg, sizeof(msg));
            	if (n <= 0) break;
            	printf("receive: %s", msg); 
        	}
        	close(fd);
    	} else { 
        	fd = open(me, O_WRONLY); 
        	while (1) {
            	fgets(msg, SMAX, stdin); 
            	int n = write(fd, msg, strlen(msg)+1); 
            	if (n<=0) break;
        	}
        	close(fd); 
    	}
    	return 0;
	}
## mmap
	#include <stdio.h>
	#include <string.h>
	#include <fcntl.h>
	#include <sys/mman.h>
	#include <unistd.h>

	#define SMAX 80

	int main(int argc, char *argv[]) {
    	int id = argv[1][0]-'0';
    	int fd = open("chat.dat", O_RDWR | O_CREAT); 
    	char *buf = mmap(NULL, 2*SMAX, PROT_READ|PROT_WRITE, MAP_SHARED, fd, 0);
    	char *myMsg, *yourMsg;
    	if (id == 0) {
        	myMsg = buf;
        	yourMsg = buf + SMAX;
    	} else {
        	myMsg = buf + SMAX;
        	yourMsg = buf;
    	}
    	if (fork() == 0) {
        	// child: receive message and print
        	while (1) {
            	if (yourMsg[0] != '\0') {
                	printf("receive: %s", yourMsg);
                	yourMsg[0] = '\0';
            	}
        	}
    	} else {
        	// parent: readline and put into myMsg in buf
        	while (1) {
            	fgets(myMsg, SMAX, stdin);
        	}
    	}
    	munmap(buf, 2*SMAX);
    	close(fd);
    	return 0;
	}
## msg
	#include <stdio.h>
	#include <string.h>
	#include <sys/types.h>
	#include <sys/ipc.h>
	#include <sys/msg.h>
	#include <sys/stat.h>
	#include <unistd.h>

	#define SMAX 80

	struct msg_t {
    	long mtype; 
    	char mtext[SMAX];
	};

	int main(int argc, char *argv[]) {
    	int id = argv[1][0]-'0';
    	int q0 = msgget((key_t) 1235, 0666|IPC_CREAT); 
    	int q1 = msgget((key_t) 1236, 0666|IPC_CREAT);
    	int myQ, yourQ;
    	if (id == 0) {
        	myQ = q0;
        	yourQ = q1;
    	} else {
        	myQ = q1;
        	yourQ = q0;
    	}
    	struct msg_t msg = {.mtype=1};
    	// char msg[SMAX];
    	if (fork() == 0) { 
        	// child: receive message and print
        	while (1) {
            	msgrcv(yourQ, &msg, SMAX, 0, 0);
            	printf("receive: %s", msg.mtext); 
        	}
    	} else {
        	// parent: readline and put into myMsg in buf
        	while (1) {
            	fgets(msg.mtext, SMAX, stdin);
            	msgsnd(myQ, &msg, SMAX, 0); 
        	}
    	}
    	return 0;
	}
## udp
	#include <stdio.h>
	#include <string.h>
	#include <stdlib.h>
	#include <sys/types.h>
	#include <sys/socket.h>
	#include <arpa/inet.h>
	#include <netinet/in.h>
	#include <unistd.h>

	#define SMAX 80

	int main(int argc, char *argv[]) {
    	int sfd = socket(AF_INET, SOCK_DGRAM, 0);
    	struct sockaddr_in saddr, raddr;
    	memset(&saddr, 0, sizeof(saddr));
    	memset(&raddr, 0, sizeof(raddr));
    	saddr.sin_family = AF_INET;
    	saddr.sin_port = htons(8888);
    	char msg[SMAX];
    	if (argc==1) { 
        	printf("I am server...\n");
        	saddr.sin_addr.s_addr = INADDR_ANY;
        	bind(sfd, (struct sockaddr*) &saddr, sizeof(struct sockaddr));
        	socklen_t rAddrLen = sizeof(struct sockaddr);
        	int rlen = recvfrom(sfd, msg, SMAX, 0, (struct sockaddr*) &raddr, &rAddrLen);
        	printf("receive: %s from client addr %s\n", msg, inet_ntoa(raddr.sin_addr));
    	} else { // client
        	printf("I am client...\n");
        	saddr.sin_addr.s_addr = inet_addr(argv[1]);
        	memcpy(&raddr, &saddr, sizeof(saddr));
        	char *connMsg = "<connect request>";
        	sendto(sfd, connMsg, strlen(connMsg)+1, 0, (struct sockaddr*) &saddr, sizeof(struct sockaddr));
    	}
    	if (fork() == 0) {
        	// child: receive message and print
        	while (1) {
            	socklen_t rAddrLen = sizeof(struct sockaddr);
            	recvfrom(sfd, msg, SMAX, 0, (struct sockaddr*) &raddr, &rAddrLen); 
            	printf("receive: %s", msg); 
        	}
    	} else {
        	// parent: readline and send msg
        	while (1) {
            	fgets(msg, SMAX, stdin);
            	sendto(sfd, msg, strlen(msg)+1, 0, (struct sockaddr*) &raddr, sizeof(struct sockaddr));
        	}
    	}
    	close(sfd);
    	return 0;
	}
## tcp
	#include <stdio.h>
	#include <string.h>
	#include <stdlib.h>
	#include <sys/types.h>
	#include <sys/socket.h>
	#include <arpa/inet.h>
	#include <netinet/in.h>
	#include <unistd.h>

	#define SMAX 80

	int main(int argc, char *argv[]) {
    	int sfd = socket(AF_INET, SOCK_STREAM, 0);
    	int cfd, fd;
    	struct sockaddr_in saddr, raddr;
    	memset(&saddr, 0, sizeof(saddr));
    	memset(&raddr, 0, sizeof(raddr));
    	saddr.sin_family = AF_INET;
    	saddr.sin_port = htons(8888);
    	char msg[SMAX];
    	if (argc==1) { 
        	printf("I am server...\n");
        	saddr.sin_addr.s_addr = INADDR_ANY;
        	bind(sfd, (struct sockaddr*) &saddr, sizeof(struct sockaddr));
        	listen(sfd, 1);
        	socklen_t rAddrLen = sizeof(struct sockaddr);
        	cfd = accept(sfd, (struct sockaddr*) &raddr, &rAddrLen);
        	printf("accept: cfd=%d client addr %s\n", cfd, inet_ntoa(raddr.sin_addr));
        	fd = cfd;
    	} else { 
        	printf("I am client...\n");
        	saddr.sin_addr.s_addr = inet_addr(argv[1]);
        	connect(sfd, (struct sockaddr*) &saddr, sizeof(struct sockaddr)); 
        	fd = sfd;
        	printf("connect success: sfd=%d server addr=%s\n", sfd, inet_ntoa(saddr.sin_addr));
    	}

    	if (fork() == 0) { 
        	// child: receive message and print
        	while (1) {
            	int n = recv(fd, msg, SMAX, 0); 
            	if (n <=0) break;
            	printf("receive: %s", msg); 
        	}
    	} else { 
        	// parent: readline and send msg
        	while (1) {
            	fgets(msg, SMAX, stdin);
            	send(fd, msg, strlen(msg)+1, 0);
        	}
    	}
    	close(sfd);
    	return 0;
	}

## RISC-V 處理器
* [RISC-V](https://zh.wikipedia.org/wiki/RISC-V)
* 指令子集
![README](./13.jpg)