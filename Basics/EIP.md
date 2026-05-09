
```
Start: stack = [puts_plt, win_address, argument]

Step 1 (call puts_plt):
  → EIP = puts_plt
  → stack becomes [win_address, argument]
  → puts(argument) executes

Step 2 (puts returns):
  → EIP = win_address (popped from stack)
  → stack becomes [argument]
  → win() executes
```