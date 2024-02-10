---
title: "MyVault Writeup"
date: 2024-02-10 +0800
categories: [CTFs Writeups]
---
الحمدلله والصلاة على رسول الله صلى الله عليه وسلم

First, we'll put the app on the emulator and see what it does.

![MyVault Main Screen](/assets/imgs/MyVault_Main_Screen.png)

There's just one screen with a place to type a 'One-Time Password (OTP)' and a 'Submit' button.

You can only type 4 numbers and hit submit. Then, a short message pops up.

![MyVault Main Screen Toast](/assets/imgs/MyVault_Main_Screen_Toast.png)

Now that we know what the app does, we're going to take apart its APK file using two tools called `apktool` and `jadx-gui` with these commands:
- `apktool d MyVault.apk`
- `jadx-gui MyVault.apk`

Next, we check the app's manifest file and confirm there's only the one screen we saw earlier.

![MyVault Manifest File](/assets/imgs/MyVault_Manifest.png)

We go into the screen's code and find it opens a file named `vault.enc`. Let's find this file and see what's inside.

![MyVault MainActivity code](/assets/imgs/MyVault_MainActivity_code.png)

It looks like the file might have something important, so we search the app for the code that locked it up.

After looking around, we find some code in the `i` package used by the main screen.

This code uses the 4 numbers we enter to try and unlock `vault.enc`, saving the unlocked content in a file named `vault.txt`. If we enter the right numbers, it shows a 'congrats' message.

![Decryption Code Snippet](/assets/imgs/MyVault_Decryption_Code_Snippet.png)

We could try all numbers from `0000` to `9999` by hand and check the `vault.txt` each time we see the 'congrats' message. But that would take forever. So, I decided to write a script to do it for us.

Here's the Python script:

```python
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import padding
import os

def decrypt_file(otp, file_path):
    key_text = otp + otp[::-1] + otp + otp[::-1]
    key = key_text.encode('utf-8')
    
    cipher = Cipher(algorithms.AES(key), modes.ECB(), backend=default_backend())
    decryptor = cipher.decryptor()
    
    with open(file_path, 'rb') as encrypted_file:
        encrypted_data = encrypted_file.read()
    
    try:
        unpadder = padding.PKCS7(128).unpadder()
        decrypted_data = decryptor.update(encrypted_data) + decryptor.finalize()
        decrypted_data = unpadder.update(decrypted_data) + unpadder.finalize()
        return decrypted_data
    except Exception as e:
        return None

def main():
    file_path = '.\\vault.enc'
    for i in range(10000):
        otp = f'{i:04d}'
        result = decrypt_file(otp, file_path)
        if result is not None:
            print(f'Success with OTP: {otp}')
            with open('vault_decrypted.txt', 'ab') as decrypted_file:
                decrypted_file.write(result)
    else:
        print("Failed to decrypt with any OTP.")

if __name__ == '__main__':
    main()
```
![MyVault Python Script](/assets/imgs/MyVault_Python_Script_Answer.png)
![MyVault Python Script](/assets/imgs/MyVault_Python_Script_Run.png)
![MyVault Python Script](/assets/imgs/MyVault_Python_Script_Result.png)

That's it! I hope you learned something new. If you have any feedback, please don't hesitate to contact me.

سبحانك اللهم وبحمدك، أشهد أن لا إله إلا أنت، أستغفرك وأتوب إليك