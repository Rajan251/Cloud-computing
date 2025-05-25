ALB doesn't have a single static IP address like traditional servers. Instead, it has **multiple dynamic IP addresses** that can change. Let me explain with examples:

## ALB IP Address Characteristics

### ALB DNS Name (What you actually get)
When you create an ALB, AWS gives you a DNS name like:
```
test-main-alb-1234567890.us-east-1.elb.amazonaws.com
```

### Dynamic IP Addresses Behind the DNS
ALB resolves to **multiple IP addresses** that change dynamically:

```bash
# Example nslookup of an ALB
$ nslookup test-main-alb-1234567890.us-east-1.elb.amazonaws.com

Non-authoritative answer:
Name:    test-main-alb-1234567890.us-east-1.elb.amazonaws.com
Address: 52.45.123.45
Address: 3.211.87.92
Address: 54.163.201.134
Address: 52.87.45.201
```

### Why Multiple IPs?
- **High Availability**: Multiple IPs across different Availability Zones
- **Load Distribution**: Traffic spread across multiple ALB nodes
- **Auto Scaling**: AWS adds/removes IPs based on traffic
- **Dynamic Changes**: IPs can change anytime (that's why you use DNS names)

## Real Examples of ALB IP Ranges

ALB IPs typically come from these AWS IP ranges (varies by region):

### US-East-1 (Virginia) Examples:
```
52.0.0.0/11     (52.0.0.1 to 52.31.255.254)
54.144.0.0/14   (54.144.0.1 to 54.147.255.254)
3.208.0.0/12    (3.208.0.1 to 3.223.255.254)
```

### US-West-2 (Oregon) Examples:
```
52.32.0.0/11    (52.32.0.1 to 52.63.255.254)
54.148.0.0/15   (54.148.0.1 to 54.149.255.254)
```

## How DNS Resolution Works

## Key Points About ALB IPs:

**❌ What ALB DOESN'T have:**
- Single static IP like `203.45.67.89`
- Fixed IP that never changes

**✅ What ALB DOES have:**
- DNS name like `test-main-alb-1234567890.us-east-1.elb.amazonaws.com`
- Multiple dynamic IPs that resolve from that DNS name
- IP addresses that change automatically

**Why AWS does this:**
- **High Availability**: Multiple IPs across different data centers
- **Auto-Scaling**: More IPs added during high traffic
- **Fault Tolerance**: If one IP fails, others still work

**What you configure in GoDaddy:**
```
test.retailer.com → test-main-alb-1234567890.us-east-1.elb.amazonaws.com
```

**NOT:**
```
test.retailer.com  → 52.45.123.45  (DON'T DO THIS!)
```

The beauty is that users never see the complex ALB DNS name - they just type your clean domain name, and DNS handles all the complexity behind the scenes!
