# AWS End User Messaging SMS - Complete Implementation Guide

## What You Need to Know

AWS End User Messaging SMS (service name: `pinpoint-sms-voice-v2`) is the **new and recommended way** to send SMS messages. SNS SMS is legacy and should not be used for new projects.

---

## Prerequisites (What You Already Have ✓)

- ✓ 10 DLC campaigns registered
- ✓ Phone number provisioned
- ✓ AWS Account with End User Messaging SMS enabled

---

## Step-by-Step Implementation Guide

### **Step 1: Gather Your Configuration Details**

Before coding, collect these details from your AWS Console:

1. **Origination Identity** (Your Phone Number)
   - Go to: AWS Console → End User Messaging SMS → Phone numbers
   - Copy your phone number (format: +1234567890)

2. **Pool ID or Configuration Set** (Optional but recommended)
   - Go to: AWS Console → End User Messaging SMS → Pools
   - Copy the Pool ID if you have one configured

3. **AWS Region**
   - Note which region you're using (e.g., `us-east-1`, `us-west-2`)

---

### **Step 2: Set Up IAM Permissions**

Create an IAM policy for your developer/application:

#### **Minimum Required IAM Policy:**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "SendSMSMessages",
            "Effect": "Allow",
            "Action": [
                "sms-voice:SendTextMessage",
                "sms-voice:SendDestinationNumberVerificationCode",
                "sms-voice:VerifyDestinationNumber"
            ],
            "Resource": "*"
        }
    ]
}
```

#### **Recommended IAM Policy (with additional monitoring):**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "SendAndMonitorSMS",
            "Effect": "Allow",
            "Action": [
                "sms-voice:SendTextMessage",
                "sms-voice:DescribePhoneNumbers",
                "sms-voice:DescribePools",
                "sms-voice:DescribeConfigurationSets",
                "sms-voice:SendDestinationNumberVerificationCode",
                "sms-voice:VerifyDestinationNumber"
            ],
            "Resource": "*"
        }
    ]
}
```

#### **How to Apply the Policy:**

1. Go to IAM Console → Policies → Create Policy
2. Choose JSON tab and paste the policy above
3. Name it: `EndUserMessagingSMSSendPolicy`
4. Attach to your application's IAM user/role

---

### **Step 3: Install Required Python Libraries**

Your developer needs to install boto3:

```bash
pip install boto3
```

Or add to `requirements.txt`:
```
boto3>=1.26.0
```

---

### **Step 4: Basic Python Code Implementation**

#### **Simple Single SMS Example:**

```python
import boto3
from botocore.exceptions import ClientError

def send_sms(phone_number, message):
    """
    Send a transactional SMS message
    
    Args:
        phone_number: Recipient's phone number in E.164 format (e.g., +1234567890)
        message: The SMS message text
    
    Returns:
        dict: Response from AWS or error details
    """
    
    # Initialize the client
    client = boto3.client(
        'pinpoint-sms-voice-v2',
        region_name='us-east-1'  # Change to your region
    )
    
    try:
        response = client.send_text_message(
            DestinationPhoneNumber=phone_number,
            OriginationIdentity='+1234567890',  # Your registered phone number
            MessageBody=message,
            MessageType='TRANSACTIONAL',  # Use TRANSACTIONAL for important messages
            DryRun=False  # Set to True for testing without actually sending
        )
        
        print(f"Message sent successfully!")
        print(f"Message ID: {response['MessageId']}")
        return response
        
    except ClientError as e:
        print(f"Error sending SMS: {e.response['Error']['Message']}")
        return None

# Example usage
if __name__ == "__main__":
    recipient = "+1234567890"  # Customer's phone number
    message_text = "Your appointment slot is booked for tomorrow at 3 PM. See you soon!"
    
    send_sms(recipient, message_text)
```

---

### **Step 5: Production-Ready Code with Error Handling**

```python
import boto3
import logging
from botocore.exceptions import ClientError, BotoCoreError
from typing import Optional, Dict

# Set up logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class SMSService:
    """Production-ready SMS service using AWS End User Messaging"""
    
    def __init__(self, region_name='us-east-1', origination_number=None):
        """
        Initialize SMS service
        
        Args:
            region_name: AWS region (e.g., 'us-east-1')
            origination_number: Your registered phone number
        """
        self.client = boto3.client(
            'pinpoint-sms-voice-v2',
            region_name=region_name
        )
        self.origination_number = origination_number
    
    def send_transactional_sms(
        self,
        destination_number: str,
        message: str,
        max_price: str = '0.50',
        ttl: int = 3600,
        dry_run: bool = False
    ) -> Optional[Dict]:
        """
        Send a transactional SMS message
        
        Args:
            destination_number: Recipient phone in E.164 format (+1234567890)
            message: SMS text content (max 160 chars for single message)
            max_price: Maximum price per message in USD (default 0.50)
            ttl: Time to live in seconds (default 1 hour)
            dry_run: If True, validates without sending
        
        Returns:
            Response dict with MessageId or None if failed
        """
        
        try:
            # Validate phone number format
            if not destination_number.startswith('+'):
                logger.error(f"Invalid phone number format: {destination_number}")
                return None
            
            # Send the message
            response = self.client.send_text_message(
                DestinationPhoneNumber=destination_number,
                OriginationIdentity=self.origination_number,
                MessageBody=message,
                MessageType='TRANSACTIONAL',  # or 'PROMOTIONAL'
                MaxPrice=max_price,
                TimeToLive=ttl,
                DryRun=dry_run
            )
            
            logger.info(f"SMS sent successfully. MessageId: {response['MessageId']}")
            return response
            
        except ClientError as e:
            error_code = e.response['Error']['Code']
            error_message = e.response['Error']['Message']
            
            # Handle specific errors
            if error_code == 'ThrottlingException':
                logger.error("Rate limit exceeded. Slow down sending.")
            elif error_code == 'ValidationException':
                logger.error(f"Invalid parameters: {error_message}")
            elif error_code == 'AccessDeniedException':
                logger.error("IAM permissions issue. Check your policy.")
            else:
                logger.error(f"Error sending SMS: {error_code} - {error_message}")
            
            return None
            
        except BotoCoreError as e:
            logger.error(f"AWS SDK error: {str(e)}")
            return None
    
    def send_slot_booking_confirmation(self, phone_number: str, slot_time: str) -> bool:
        """
        Example: Send appointment booking confirmation
        
        Args:
            phone_number: Customer's phone number
            slot_time: Appointment time
        
        Returns:
            True if sent successfully, False otherwise
        """
        message = f"Your appointment slot is confirmed for {slot_time}. Thank you!"
        
        response = self.send_transactional_sms(
            destination_number=phone_number,
            message=message
        )
        
        return response is not None
    
    def send_reminder(self, phone_number: str, reminder_text: str) -> bool:
        """
        Example: Send reminder message
        
        Args:
            phone_number: Customer's phone number
            reminder_text: Reminder message
        
        Returns:
            True if sent successfully, False otherwise
        """
        message = f"Reminder: {reminder_text}"
        
        response = self.send_transactional_sms(
            destination_number=phone_number,
            message=message
        )
        
        return response is not None


# Example usage
if __name__ == "__main__":
    # Initialize the service
    sms_service = SMSService(
        region_name='us-east-1',
        origination_number='+1234567890'  # Replace with your number
    )
    
    # Send booking confirmation
    success = sms_service.send_slot_booking_confirmation(
        phone_number='+19876543210',
        slot_time='Tomorrow at 3:00 PM'
    )
    
    if success:
        print("Booking confirmation sent!")
    else:
        print("Failed to send confirmation")
    
    # Send reminder
    sms_service.send_reminder(
        phone_number='+19876543210',
        reminder_text='Your appointment is in 1 hour'
    )
```

---

### **Step 6: Configuration Using Environment Variables (Recommended)**

Create a `.env` file:

```env
AWS_REGION=us-east-1
AWS_SMS_ORIGINATION_NUMBER=+1234567890
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
```

Updated code with environment variables:

```python
import os
from dotenv import load_dotenv

# Load environment variables
load_dotenv()

# Initialize service with env vars
sms_service = SMSService(
    region_name=os.getenv('AWS_REGION'),
    origination_number=os.getenv('AWS_SMS_ORIGINATION_NUMBER')
)
```

---

### **Step 7: Important boto3 API Parameters Explained**

#### **Required Parameters:**
- `DestinationPhoneNumber`: Recipient's phone (E.164 format: +1234567890)
- `OriginationIdentity`: Your registered phone number or Sender ID
- `MessageBody`: The SMS text (max 1600 chars, but splits into multiple parts)

#### **Important Optional Parameters:**
- `MessageType`: 
  - `TRANSACTIONAL` - For important messages (OTP, confirmations, reminders)
  - `PROMOTIONAL` - For marketing messages
- `MaxPrice`: Maximum price willing to pay per message (USD, default: 0.50)
- `TimeToLive`: How long to retry delivery (seconds, max: 259200 = 72 hours)
- `DryRun`: Set to `True` to test without actually sending
- `ConfigurationSetName`: For tracking delivery status (optional)
- `Context`: Key-value pairs for custom data (optional)

---

### **Step 8: Testing Your Implementation**

#### **Test 1: Dry Run (No actual SMS sent)**

```python
response = sms_service.send_transactional_sms(
    destination_number='+1234567890',
    message='Test message',
    dry_run=True  # Only validates, doesn't send
)
```

#### **Test 2: Send to Your Own Number**

```python
response = sms_service.send_transactional_sms(
    destination_number='+1234567890',  # Your number
    message='This is a test message from AWS End User Messaging'
)
```

---

### **Step 9: Common Error Codes and Solutions**

| Error Code | Meaning | Solution |
|------------|---------|----------|
| `AccessDeniedException` | IAM permissions missing | Add required IAM policy |
| `ThrottlingException` | Sending too fast | Implement rate limiting |
| `ValidationException` | Invalid parameters | Check phone format, message length |
| `ResourceNotFoundException` | Phone number not found | Verify your origination number |
| `OptedOutException` | Recipient opted out | Remove from your list |

---

### **Step 10: Monitoring and Costs**

#### **Check Delivery Status:**

You can optionally configure a Configuration Set to get delivery reports via CloudWatch or SNS.

#### **Pricing:**
- Transactional SMS: ~$0.00645 per message (US)
- Costs vary by country
- Check current pricing: [AWS Pricing](https://aws.amazon.com/end-user-messaging/pricing/)

#### **Monitor Spending:**

```python
# Set spending limits in AWS Console
# End User Messaging SMS → Account settings → Spending quotas
```

---

## Quick Start Checklist for Your Developer

- [ ] Install boto3: `pip install boto3`
- [ ] Get origination phone number from AWS Console
- [ ] Apply IAM policy to user/role
- [ ] Configure AWS credentials (access key or IAM role)
- [ ] Copy the production-ready code above
- [ ] Replace `origination_number` with your phone number
- [ ] Replace `region_name` with your AWS region
- [ ] Test with `dry_run=True` first
- [ ] Send test SMS to your own number
- [ ] Implement error handling and logging
- [ ] Deploy to production

---

## Additional Resources

- [Official AWS SMS Documentation](https://docs.aws.amazon.com/sms-voice/)
- [Boto3 API Reference](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/pinpoint-sms-voice-v2.html)
- [AWS End User Messaging Console](https://console.aws.amazon.com/sms-voice/)

---

## Support

If you encounter issues:
1. Check CloudWatch logs for errors
2. Verify IAM permissions
3. Confirm phone number is registered and active
4. Check AWS Service Health Dashboard
5. Contact AWS Support if needed
