Zelix Klassmaster 13's opaque jumps are made up of two things:
* A boolean public static final field with a name consisting of whitespace characters which is never assigned a value other than its default: false
* Each method gets the value of this field, stores it in a local variable, and makes "opaque" jumps based on this value (using `ifne`)

These jumps are obviously not very opaque as we can gather from simple analysis, and it is very easy to remove them by replacing all of the variable loads with a false expression.

Before (Krakatau):
```Java
    final public static boolean \u2004\u2006\u2007\u2006\u2008\u2007\u2003\u2002\u2005;
    public String 1(String s) {
        boolean b = \u2004\u2006\u2007\u2006\u2008\u2007\u2003\u2002\u2005;
        label1: {
            Exception a = null;
            if (b) {
                break label1;
            }
            if (b) {
                break label1;
            }
            label0: {
                String s0 = null;
                try {
                    byte[] a0 = this.0(org.apache.commons.codec.binary.Base64.decodeBase64(s));
                    if (b) {
                        break label1;
                    }
                    if (b) {
                        break label1;
                    }
                    s0 = new String(a0);
                } catch(Exception a1) {
                    a = a1;
                    break label0;
                }
                return s0;
            }
            a.printStackTrace();
            if (!b && !b) {
                String s1 = null;
                return s1;
            }
        }
        String s2 = null;
        return s2;
    }
 ```
 After:
 ```Java
    public String 1(String s) {
        String s0 = null;
        try {
            s0 = new String(this.0(org.apache.commons.codec.binary.Base64.decodeBase64(s)));
        } catch(Exception a) {
            a.printStackTrace();
            String s1 = null;
            return s1;
        }
        return s0;
    }
```
