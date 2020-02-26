### String Char 정리
#### String
* ```string :: npos```
* string[0] 배열식으로 접근 가능
* ```string.compare(string) ```:: abc 선후 관계 파악 가능
* 문자 to int
   * string to int: ```atoi(string)```
   * char to int: ``` int (char) ```
* int to 문자
    * ``` to_string(int) ``` : 10 -> 10
    * ``` char (int) ```  ?? 안된다
* string to char ``` char * string.c_str() ```
* char to string
   * ```string k // string k = 'a' (X) ```
   * ```k = 'a'; ```
 * ```string.insert(index, input); // index 위치에 글자 없어지지 않음```
