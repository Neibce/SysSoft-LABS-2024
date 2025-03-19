# BombLab
- Defusing A Binary Bomb
- [Writeup](https://github.com/Neibce/SysSoft-LABS-2024/blob/main/1.%20BombLab/Bomblab_writeup.pdf)
  
## 1. Phase 1
![image](https://github.com/user-attachments/assets/dab83c5a-7414-4930-b242-0bcec651adcc)<br>
phase_1 함수를 보면 strings_not_equal 함수 호출을 통해 문자열 비교가 이루어짐을 추측할 수 있고, test, jne를 보아 함수의 return value가 0이 아닐 경우 explode임을 알 수 있다.<br>
![image](https://github.com/user-attachments/assets/2f22b2b7-4b51-4c62-b154-13c910a7d99c)<br>
strings_not_equal 함수를 호출하기 전에 lea로 rsi에 할당되는 값(0x2bd0)의 내용을 조회했더니 답인 “You can Russia from land here in Alaska.”을 알 수 있었다.<br>

## 2. Phase 2
![image](https://github.com/user-attachments/assets/6d9d4168-2039-43cb-a42c-89a5cf426243)<br>
&nbsp;먼저 스택 포인터인 rsp를 rsi로 옮기고, read_six_numbers를 실행한 뒤 rsp 위치의 값이 0보다 작으면 explode_bomb으로 jump 하는 것으로 보아 read_six_numbers는 2번째 arg로 배열의 주소를 받아 입력 받은 6개의 숫자를 채우는 함수로 추측하였고 read_six_numbers 함수를 disass한 결과 sscanf와, sscanf 실행 전 lea 명령으로 sscanf의 arg에 rsi에서 4의 배수로 더해진 값들을 넣는 것을 보고 확신하게 되었다. js를 통해 첫번째 숫자는 0 이상이기만 하면 됨을 알아내었다.<br>
![image](https://github.com/user-attachments/assets/d50d81b6-1c71-40f6-9f76-edbed84a0456)<br>
이후 숫자들은 어셈블리를 해석하면서(주석 참고) rbx로 1부터 5까지 5번 도는 반복문임을 알았고, 통과 조건은 arr[rbx - 1] + ebx == arr[rbx] 임을 알아냈다. 즉, 1,2,3,4,5씩 커지면 되는 것이다. 0,1,3,6,10,15를 입력했더니 통과되었다.<br>

## 3. Phase 3
![image](https://github.com/user-attachments/assets/4d35a297-35df-4dbf-87fe-ba4eadfd2455)<br>
![image](https://github.com/user-attachments/assets/4c755612-54d9-4d16-8fc6-c29a7f12fd5c)<br>
먼저 phase3 내부 sscanf의 입력을 알아보고 싶어 rsi에 들어가는 0x2c26의 내용을 print하였고,
“%d %c %d” 형식으로 입력 받음을 알 수 있었다. sscanf 전의 lea 명령어들을 통해 rsp+0xf에 char, rsp+0x10에
첫번째 int, rsp+0x14에 마지막 int를 입력 받음을 알 수 있다.<br>
![image](https://github.com/user-attachments/assets/fc552be0-34da-42f1-bcd2-6f085b63f38a)<br>
첫번째 입력(n1) 값에 따라 jump table을 통해 jump(switch-case)함을 알 수 있다.<br>
jump table을 확인해보면,&nbsp;
<img width="100" alt="image" src="https://github.com/user-attachments/assets/b7887af7-7bad-4ff7-8950-b794e4b42891" />&nbsp;
<img width="100" alt="image" src="https://github.com/user-attachments/assets/d9900b94-d505-4377-af7b-8bea6505426f" />&nbsp;
<img width="100" alt="image" src="https://github.com/user-attachments/assets/092c9344-b20d-482c-8961-81b409690dd2" /><br>
n1|pos|value|goto (value+0x2c40)
:--:|:--:|:--:|:--:
0|0x2c40|0xffffe76c|0x13ac
1|0x2c44|0xffffe78e|0x13ce
2|0x2c48|0xffffe7b0|0x13f0

![image](https://github.com/user-attachments/assets/956321af-235e-4659-a54a-3e540584f36e)<br>
13ac부터 확인해보면, 0x254(596)과 마지막 입력 값이 같아야 explode를 피할 수 있음(je 0x14a0)을 알 수 있다.
![image](https://github.com/user-attachments/assets/e21fbd82-a0a1-4c08-bdc2-f72f61f4fc43)<br>
14a0을 보면, al과 입력받은 문자가 같아야 explode를 피할 수 있다. 13ac에서 0x78(120)을 eax에 mov했으므로 x를 입력해야 됨을 알 수 있다. 나머지 jump table은 무시하고 0 x 596 을 입력했더니 통과되었다.<br>

## 4. Phase 4
sscanf 직전의 rsi 값을 확인하니 입력이 2개의 정수(%d %d)임을 알 수 있었다.
![image](https://github.com/user-attachments/assets/d19dd12c-fd87-43bf-87d9-f7a0a1d4c9ff)<br>
rsp에 첫번째 정수(n1), rsp+4에 두번째 정수(n2)를 입력받고 있다.<br>
![image](https://github.com/user-attachments/assets/e969f739-ff9a-4aef-93cf-33fb23508556) 을 통해 첫번째 정수는 0xE(14) 이하여야 함을 알 수 있다.<br>
![image](https://github.com/user-attachments/assets/d84fc894-999e-4bb7-b1e6-ac882d2f845a)<br>
이후 코드를 분석해보면 func4(n1, 0, 14)의 결과가 4이고, n2 값이 4여야 통과됨을 찾을 수 있다.<br>
func4를 분석해보았더니 이분탐색 느낌의 재귀함수가 나왔다.<br>
![image](https://github.com/user-attachments/assets/330603be-1b4a-4887-a859-d7e5cea8f382)<br>
![image](https://github.com/user-attachments/assets/e2f6f0a7-8c12-40e7-8224-1925bfb0c8fc)<br>
n1에 뭘 넣어야 4가 return 될지 바로 알기 어려워서 c로 해당 함수를 똑같이 구현해서 1부터 14까지 넣어보았고,
2를 넣었을 때 4가 return됨을 알 수 있었다. 따라서 2 4를 입력했더니 통과되었다.

## 5. Phase 5
![image](https://github.com/user-attachments/assets/05dc7cd0-8559-4fe4-b818-bae6320a02ad)<br>
&nbsp;phase_5 함수를 분석했더니 입력 문장의 길이가 6이여야하고, 입력된 문자열들의 글자를 하나씩 뽑아 하위 4비트만 남긴 뒤 0x2c60에 위치하는 문자열에서 그에 해당하는 번째의 문자를 가져와 (0x2c60[input_str_ptr[rax] & 0xf]) 배열에 저장한 뒤, string_not_equal이라는 함수를 통해 0x2c2f에 있는 devils라는 문자열과 비교하고 있음을 알 수 있었다.<br>
&nbsp;0x2c60에 위치한 문자열은 “maduiersnfotvbylSo you think…”인데, 0x2c60[input_str_ptr[rax] & 0xf]이기 때문에 16글자 이상 가져올 수 없다. devils를 만드려면 0x2 0x5 0xC 0x4 0xF 0x7번째 문자들을 가져와야 한다. 하위 4비트만 일치하면 되므로 아스키 코드표를 보고 해당하는 BELDOG를 입력했더니 통과되었다.<br>

## 6. Phase 6
![image](https://github.com/user-attachments/assets/6ac96cb0-f10e-4be9-9c15-89b974ecc6b1)<br>
위 부분을 통해 6개의 정수를 r13(rsp)에 배열로 입력 받고 있음을 알 수 있었다.<br>
![image](https://github.com/user-attachments/assets/c56e712e-aefe-473f-963d-f90532e48d7d)<br>
&nbsp;차례대로 해석하다 보니 input 배열에서 6 이상의 값이 있을 경우 explode가 됨을 알 수 있었고, rbp와 r13을 증가시키면서 서로 비교하는 부분을 확인 할 수 있었다. 즉, 입력 받은 값들 중 중복이 있으면 explode 하는 2중 반복문임을 확인할 수 있었다. 이를 통해 입력에 중복된 값이 있으면 안되는 것을 알게 되었다.<br>
![image](https://github.com/user-attachments/assets/8b970c4d-2dd8-47f4-946f-dec38120fbcd)<br>
계속 해석해 나가다 보니 하나의 반복문이 더 존재했는데, input array의 주소를 저장해 두었던 r12를 증가시키면서 array의 6개 원소 값에 각각 7–(기존 값) 한 것을 저장하고 있음을 알 수 있었다.<br>
![image](https://github.com/user-attachments/assets/c01e7c34-bc63-4212-b0a6-07baacd67c8f)<br>
&nbsp;이후 2중 반복문을 확인할 수 있었는데, ppt에 phase 6이 linked list/structs라는 힌트를 보고 처음 16be에 lea를 통해 rdx에 0x204230을 넣는 부분에 gdb에서 node1이라는 주석을 달았길래 struct에 관련된 부분일거라 추측했고, 
0x204230를 읽어보니 <img width="200" alt="image" src="https://github.com/user-attachments/assets/5edc846f-0d0e-4cb9-9c75-9a1aeca2045c" /> 라는 값이 나와 struct라고 가정하고 계속 진행했다.<br>
&nbsp;내부 2번째 반복문에서 0x204230+8 에 접근하고, 0x204230+8 에 저장된 주소에 접근하는 방식임을 알았고, linked list일거라 생각되어 0x204230+8, *(0x204230+8), *(*(0x204230+8)+8)…에 접근해봤더니 6개짜리 linked list임을 알 수 있었다.<br>

node|+0|+8|node|+0|+8
:--:|:--:|:--:|:--:|:--:|:--:
1|0x148(328)|0x204240|4|0x2ba(698)|0x204270
2|0x16a(362)|0x204250|5|0x327(807)|0x204110
3|0x358(856)|0x204260|6|0x2d4(724)|0x0

![image](https://github.com/user-attachments/assets/320a5f41-b46a-43b8-b434-86468f2f9d12)<br>
따라서 이 부분의 코드는 이전에 (7-입력 값) 했던 배열에서 수들을 차례로 가져와 (7-입력 값)번째의 node 구조체의 주소를 새로운 배열(rsp+32)에 저장하고 있음을 알 수 있다.<br>
![image](https://github.com/user-attachments/assets/b0394532-e460-49ba-a866-0893a9d1b7bb)<br>
반복문 실행 이후 코드를 해석해보니 (rsp+32) 배열에 넣은 node의 순서대로 node들의 2번째 값(다음 node의 주소)을 업데이트 하고 있음을 알 수 있었다.
![image](https://github.com/user-attachments/assets/a1268f3c-dff7-4540-a2b6-10ff4ea9a300)<br>
이후 반복문이 하나 더 존재했는데, node들을 순차적으로 확인하면서 앞의 node가 가지고 있는 값(+0)이 뒤의 노드가 가지고 있는 값보다 커야 explode가 되지 않으므로 아까 확인한 값들을 사용하면, node3, 5, 6, 4, 2, 1 순이어야 함을 알 수 있다. 아까 7-(입력 값) 하는 코드가 있었으므로 답은 4 2 1 3 5 6이다.<br>

## 7. Phase Secret
먼저, secret_phase 함수를 실행할 방법을 찾기 위해 main에서 실행되는 함수들을 모두 disass해서 secret_phase에 대한 내용이 있는 함수를 보았더니 phase_defused 함수에 있는 것을 찾을 수 있었다.<br>
![image](https://github.com/user-attachments/assets/07a24546-70dd-4f96-8693-501a1d7df936)<br>
이 부분을 통해 일단 0x5555557586의 값이 6이 되어야 함을 알 수 있었고, 이 값이 의미하는 바를 알기 위해 phase_defuse+30에 bp를 걸었다. phase1을 성공했을 때는 1, 2를 성공했을 때는 2 이런 식으로 값이 되는 것을 보아 일단 폭탄 6개를 다 해체 한 후에 실행되는 부분임을 알 수 있었다.<br>
![image](https://github.com/user-attachments/assets/969d90ea-cb50-4ad3-8f96-6e3a59cd6e69)<br>
이후 sscanf를 호출하고 있었는데, input(rdi)을 어디서 가져오는지 보기 위해서 <img width="240" alt="image" src="https://github.com/user-attachments/assets/65dd638a-5348-4252-aa89-4512076e17bd" /> 명령을 실행했고, phase 4때의 입력 값이랑 동일함을 확인 할 수 있었다.<br>
<img width="190" alt="image" src="https://github.com/user-attachments/assets/e044b2ea-970b-48d3-83ae-96f0e3fda450" /> 명령을 통해 phase 4 입력 값 뒤에 문자열을 하나 더 확인하고 있음을 보았고, 그 값은 r8인 (rsp+0x10)에 저장됨을 알 수 있다.<br>
![image](https://github.com/user-attachments/assets/442169db-c945-40d2-9bba-a59259d4d5a9)<br>
이 부분에서 (rsp+0x10)의 문자열과 <img width="168" alt="image" src="https://github.com/user-attachments/assets/126f6e3e-f89c-4272-adde-4d1cb1ec01b8" />를 비교하고 있음을 알았고, secret_phase의 실행 조건은 phase_4의 입력 뒤에 DrEvil을 덧붙이는 것임을 알 수 있었다.<br>
![image](https://github.com/user-attachments/assets/e9f6f542-80fb-420e-b8ca-37ab959a54b6)<br>
이제 secret_phase를 disass 해보았는데, 한 줄을 읽고 strtol을 통해 10진수 숫자 1개를 얻음을 알 수 있었고, 그 값에서 1을 뺀게 0x3E8(1000)보다 클 경우 explode임을 알 수 있었다.<br>
![image](https://github.com/user-attachments/assets/369de946-0ee3-4f2a-b3f4-bad490821a60)<br>
이후 부분에서 fun7(0x555555758150, input_num)을 호출하는 부분이 있었고, 리턴 값이 7이 아니면 explode임을 확인 할 수 있었다.<br>
n1의 값은 <img width="213" alt="image" src="https://github.com/user-attachments/assets/1eed61bf-6e09-45c5-9592-d8f17dd2101f" />(36)임을 확인했으므로 fun7을 disass해보았다.<br>
![image](https://github.com/user-attachments/assets/ddb9234e-b9c4-4459-96e6-d464a98d4eea)<br>
fun7을 해석해보니 arg1이 입력 값보다 크면 return 2 \* fun7(\*(rdi+8), input_num)을, 다르면(즉, 작으면) return 2 \* fun7(\*(rdi+16), input_num) +1을 실행, 같으면 return 0임을 확인할 수 있었다. 최종 return값이 7이 되려면 2\*fun7(..)+1이 3번 실행되면 되므로 (2\*(2\*(2\*0+1)+1)+1 = 7) 입력 값은 \*(\*(\*(rdi+16)+16)+16)의 값과 같으면 된다.<br>
<img width="170" alt="image" src="https://github.com/user-attachments/assets/8d770249-6533-45ed-a47c-62904c4293a4" /> 해당 값은 1001이므로 답은 1001이다.



