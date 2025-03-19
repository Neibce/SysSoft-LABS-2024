# AttackLab

## 1.	Phase 1
![image](https://github.com/user-attachments/assets/7834783e-8406-4df4-96ef-9a890712cf1d)<br>
getbuf 함수를 보면 0x28만큼 스택에서 할당받고 있음을 알 수 있고, buffer overflow를 통해 touch1 함수 실행을 유도하려면 0x28만큼을 다 채운 후 touch1 함수의 위치를 적어주면 된다.<br>
![image](https://github.com/user-attachments/assets/a770bc4f-e1af-456c-97f0-7909ff25315a)<br>
touch1 함수의 위치가 0x401859이므로 답은<br>
![image](https://github.com/user-attachments/assets/f6f1b7a2-c24a-4baf-a905-4319978aadb8)이다. 맨 하단의 주소 값은 little-endian 형식으로 작성하였다.<br>

## 2. Phase 2
Writeup에 제공된 소스코드를 보면 1번째 매개변수인 val이 cookie와 일치할 경우 통과임을 알 수 있다.
![image](https://github.com/user-attachments/assets/d60bf24f-52e1-4be4-b854-b8d1726e28f1)<br>
val은 touch2 함수 실행 시 rdi에 들어있는 값이므로 touch2 함수 실행 전에 rdi에 cookie 값을 넣어주면 된다.<br>
![image](https://github.com/user-attachments/assets/564bd98a-434a-43bc-89a4-e58cd7aff366)<br>
해당 코드를 작성해 gcc, objdump로 해당 코드를 byte형식으로 변환할 수 있었다.<br>
![image](https://github.com/user-attachments/assets/fea5b16a-9559-410c-835c-9e2c51190100)<br>
Gets 된 문자열이 어디 저장되는지 알기 위해 Gets 함수 실행 후 바로 다음 줄에 bp를 걸고 rsp를 확인해 찾을 수 있었다.<br>
![image](https://github.com/user-attachments/assets/e98c03c2-4fb0-4287-a05b-ae064a67cdf2)
![image](https://github.com/user-attachments/assets/a0ab1591-39e0-44e3-81f2-a32d1ce31b02)<br>
따라서 답은 아래와 같다. 1번째 줄은 rdi에 cookie 값을 넣는 명령이고, 이후 줄은 buffer overflow를 위한 의미없는 내용들, 끝에서 2번째 줄은 1번째 줄에 있는 명령을 실행하기 위해 아까 얻어낸 Gets된 문자열의 시작 위치(rsp)를 적었다. 마지막 줄은 touch2 함수를 실행하기 위한 위치이다.
![image](https://github.com/user-attachments/assets/f7db0509-2ef8-44e8-a5ec-d24378d9552b)
![image](https://github.com/user-attachments/assets/451cbbe9-dcad-4188-b92c-ef6b41f2c629)<br>

## 3. Phase 3
![image](https://github.com/user-attachments/assets/1296d99b-a2da-4693-a0e9-dcd0b09e49c4)<br>
2번과 거의 비슷한 방법으로 해결할 수 있었다.<br>
&nbsp;Writeup에 제공된 c코드를 통해 cookie값(16진수)를 문자로 변환해서 비교하고, 같을 경우 통과하는 내용임을 알아내었고. 따라서 touch3(“53374143”)을 실행해야 함을 알았다.<br>
&nbsp;2번의 답을 가져와서 마지막 줄에 있던 touch2의 위치를 touch3의 것으로 바꾸어주고, 맨 아래에 Cookie 값을 문자열로 변환한 것(“53374143”)을 추가해 주었다. 맨 아래에 Cookie 값을 넣은 이유는 Writeup의 내용 중 hexmatch와 strncmp 함수 호출 시 스택에 푸시하면서 원래 내용이 덮어씌워지는 것에 주의하라는 내용이 있어 가장 아래 쪽에 Cookie값을 한번 추가해보았다.<br>
&nbsp;rdi에는 Cookie값을 문자열로 변환한 것의 위치가 들어가야 하므로(0x556353a8+0x38) 다시 gcc와 objdump를 통해 해당 기능을 하는 코드를 첫번째 줄에 추가해 주었더니 해결할 수 있었다.<br>
![image](https://github.com/user-attachments/assets/8155d992-5ed4-4b38-a842-7fc5c701600e)
![image](https://github.com/user-attachments/assets/6228997a-59c7-4e6e-a4b5-868cb50d559c)<br>

## 4. Phase 4
Phase 2 문제와 내용이 동일해 rdi에 cookie 값을 넣어야 함을 알 수 있었다. Phase 2와는 다르게 스택에서 코드를 실행할 수 없으므로 스택에 cookie값을 넣어두고 pop으로 꺼내는게 최선일 것 같았다. Writeup 내용을 보고 popq %rdi 가젯을 objdump를 통해 찾아보았으나 찾을 수 없었고, 401a49에서 popq %rax만 찾을 수 있었다.<br>
![image](https://github.com/user-attachments/assets/b80e6d8e-9a9c-4986-85f5-5c76d73057c6)<br>
movq %rax, %rdi가 있는지 찾아보았고, 401a54에서 찾을 수 있어서 popq %rax 후 mov를 하기로 했다.<br>
![image](https://github.com/user-attachments/assets/596c3c27-bea9-445a-8f1e-f100045b2eb9)<br>
![image](https://github.com/user-attachments/assets/efcf3fd5-0d2c-47f1-81f3-96dd97d10ed6)<br>
첫 5줄은 buffer overflow를 위한 의미없는 내용들이고, 6번째 줄은 위에서 찾은 popq의 위치이다. cookie 값이 pop 되어야 하므로 다음 줄에 cookie의 값을 넣어주었다. 이후 찾았던 movq의 위치를 적어주었고, 마지막에는 Phase 2에서 찾았던 touch2 함수의 위치를 적어주니 잘 통과할 수 있었다.<br>

## 5. Phase 5
&nbsp;Phase 3과 같이 touch3 함수를 실행해야 되는 것을 writeup을 보고 알았고, 그러므로 똑같이 rdi에 “53374143” 문자열의 주소를 넣어주면 된다. 하지만 Phase 3과는 다르게 실행 전에 스택 주소를 특정할 수 없으므로 rsp의 값을 읽어와 문자열이 있는 곳을 가르키도록 값을 더해서 rdi에 저장하는 방법으로 해결 할 수 있을 것 같았다.<br>
&nbsp;start_farm과 end_farm 사이에서 rsp를 mov하는 가젯들을 찾아보았는데, movq %rsp, %rax(48 89 e0)만 존재해(401b11) 어쩔 수 없이 rsp를 rax에 mov하고 movq rax, rdi 명령을 찾아 rsp를 rdi로 mov했다. (rsp -> rax -> rdi)<br>
![image](https://github.com/user-attachments/assets/078734b1-42e2-4ca3-8dc6-bfa645ae591c) (0x401b11 movq %rsp, %rax)<br>
![image](https://github.com/user-attachments/assets/ee3dcb53-ef61-4c17-b0b4-cf0afe579234) (0x401a54 movq %rax, %rdi)<br>
이후 얻어온 rsp(rdi)값에 맨 마지막에 들어갈 cookie의 위치까지의 차만큼 더해주어야 하는데, 마침 farm 구간 내에 add_xy라는 2개의 파라미터를 읽어 더한 값을 리턴하는 함수를 찾을 수 있었다.<br>
![image](https://github.com/user-attachments/assets/bfadfa42-3651-453b-b0ea-b8ef809a582e)<br>
&nbsp;add_xy의 파라미터로 아까 가져온 rsp값과 cookie 문자열까지의 거리를 넣어 더해주면 된다. rsp값은 이전에 rdi로 옮겼으므로 cookie까지의 거리만 rsi에 넣어주면 된다. pop 명령어를 통해 원하는 값(cookie까지의 거리)을 rsi에 넣어주려 했으나, pop %rax만 존재해(0x401a49) pop이후 eax를 esi까지 eax->edx->ecx->esi를 거쳐 옮겨주었다. (eax에서 esi로 옮기는 명령을 찾을 수 없어 거쳐서 옮기게 되었다.) (거리값은 임의로 지정해두고, 나머지 명령을 다 작성한 뒤 거리가 72(0x48)임을 계산할 수 있었다.)<br>
![image](https://github.com/user-attachments/assets/9a7126b2-1b46-434a-a52e-5190242b62b2) (0x401a49 pop %rax)<br>
![image](https://github.com/user-attachments/assets/09bef896-efb6-4bb4-91a8-a0f657384bee) (0x401ab4 movl %eax, %edx)<br>
![image](https://github.com/user-attachments/assets/d80fccdd-16c8-4740-ac14-8a7ea4501180) (0x401a72 movl %edx, %ecx)<br>
![image](https://github.com/user-attachments/assets/2f015024-034b-4829-9735-4b991105f27d) (0x401a78 movl %ecx, %esi)<br>
이후 add_xy를 실행하고, return 값(rax)를 아까 찾은 0x401a54의 movq %rax, %rdi로 rdi에 옮긴 뒤, touch_3을 실행했더니 통과되었다.
![image](https://github.com/user-attachments/assets/a9467451-e253-4c59-b675-aac7134f4aba)

















