1.数组接收信息

- 保证能在收的地方后面加、0

```C
char buffer[6] = {0};
char *mystring(){
    char *s = "Hello world";
    for(int i = 0;i < (sizeof(buffer) - 1); i++){
    	buffer[i] = *(s+i);
    }
    return buffer;
}
```

