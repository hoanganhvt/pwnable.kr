It would be extremely slow if you execute it from you local computer so you should ssh to the pwnable server and solve it.
```
from pwn import *
import re

def getNum(s:bytes):
	arr=[0,0]
	cur_idx=0
	for i in range(len(s)):
		if(s[i]==32):
			cur_idx+=1
			continue
		if(s[i]>=48 and s[i]<=57):
			arr[cur_idx]=arr[cur_idx]*10+s[i]-48
	return arr

def genString(first,last):
	S=''
	for i in range(first,last+1):
		S=S+str(i)+' '
	return S
io=remote("localhost",9007)
io.recv()

for t in range(100):
	l=io.recv()
	arr=getNum(l)


	N=arr[0]
	C=arr[1]

	first=0
	last=N-1

	for i in range(C):
		mid=(first+last)//2
		io.sendline(genString(first,mid))
		res=io.recv()
		res=int(res)
		if(res%2!=0):
			last=mid 
		else:
			first=mid+1
	io.sendline(str(first))
	print(io.recv())


print(io.recv())
```
