1.level2

```

void unsafe() {
    char buffer[64];

    puts("Overflow me:");
    gets(buffer);  // Intentional vulnerability
    
    void *ret_addr = __builtin_return_address(0);
    printf("Return address: %p\n", ret_addr);
}
```
we need to overflow the buffer and also need to overflow the address and we will go to the win 

```
void win(unsigned int param1, unsigned int param2) {
    puts("Win function is running.");
    if (param1 == 0xDEADBEEF && param2 == 0xC0DEBEEF) {
        puts("Exploited!!!!!");
    } else {
        puts("Where are the parameters?? :(");
    }
}

```




