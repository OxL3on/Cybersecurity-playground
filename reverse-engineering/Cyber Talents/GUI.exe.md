# GUI.exe Writeup

The challenge provided a `.NET` executable:

```text
GUI.exe: PE32 executable, Mono/.Net assembly
```

Since it was a .NET binary, I used [ILSpy](https://github.com/icsharpcode/ILSpy?utm_source=chatgpt.com) / `ilspycmd` to decompile the executable and inspect the source code.

Inside the `button1_Click()` function, the application checked whether:

```csharp
int.Parse(label3.Text.Split(' ')[current]) + 10 == num
```

The hidden values were stored in `label3.Text`:

```text
92 98 87 93 113 95 105 85 106 94 95 105 85 89 87 91 105 87 104 85 89 95 102 94 91 104 53 115
```

This meant the real flag characters were simply:

```text
stored_value + 10
```

I wrote a small Python script to decode them:

```python
nums = [92,98,87,93,113,95,105,85,106,94,95,105,85,89,87,91,105,87,104,85,89,95,102,94,91,104,53,115]

print(''.join(chr(x + 10) for x in nums))
```
