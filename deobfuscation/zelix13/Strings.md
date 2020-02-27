Zelix's string obfuscation is very powerful, and I found it very difficult to even attempt to statically reverse it.

It consists of two layers of decryption functions. These functions do not rely on the state of any class other than their own.
Because of this they are incredibly easy to virtualise. 
This is the most simple way, but if I have time I will try to properly reverse the string obfuscation allowing much faster deobfuscation.
