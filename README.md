# Auto-Create-Oracle-Always-Free-ARM-VM-with-Telegram-Notification
Auto-Create Oracle Always Free ARM VM with Telegram Notification

This script automatically retries creating an Oracle Cloud ARM VM in the Always Free tier until capacity is available, then sends a Telegram message upon success.
<hr>

Useful Hints: <br>
<img width="700" alt="image" src="https://github.com/user-attachments/assets/b99a1cc4-f233-4653-aac2-b3adbf1069dc" /> 
![image]([https://github.com/user-attachments/assets/71dfeb58-358b-4493-b04f-8099a3a51248](https://github.com/user-attachments/assets/b99a1cc4-f233-4653-aac2-b3adbf1069dc))


<h3>✅ What This Script Will Do </h3> 
   - Use Ubuntu 22.04 LTS (ARM) image. <br>
    - Try to create an instance of VM.Standard.A1.Flex with: <br>
        - 4 OCPUs <br>
        - 24 GB memory (Always Free limit) 
        - 200 GB Storage<br>
    - Retry in loop if capacity error (out of host capacity). <br>
    - Rotate through ADs: AD-1, AD-2, AD-3. <br>

<hr>

FYI: I used GCP VM to run this script. Why not Oracle itself? If your not going to use all the limit to create this one VM, well you can run the script in Oracle VM itself [Choice is yours] <br>

**<h3><h2>✅ GCP VM Setup Quick Guide (Free Tier) </h2>** <br>
- Go to: https://console.cloud.google.com/compute/instances <br>
- Click: “Create Instance” <br>
- Choose: <br>
- Region: `us-west1` or any Free Tier region <br>
- Machine type: `e2-micro` (Free Tier eligible) <br>
- Boot disk: `Ubuntu 22.04 LTS – x86/64`, amd64 (recommended) <br>
- Firewall: Allow HTTP, Allow HTTPS traffic <br>
- Create the instance <br>
- Open the Instance with SSH - Open in browser window <br></h3>

<img width="535" alt="image" src="https://github.com/user-attachments/assets/84b8ff43-c434-4f06-a61c-485dcff5edc1" /> 
<img width="538" alt="image" src="https://github.com/user-attachments/assets/e83ecebc-17e7-4314-90b0-708bb1e32f56" />
<img width="532" alt="image" src="https://github.com/user-attachments/assets/0e0b4987-d307-432b-8736-bde70cda0fe7" />
<img width="537" alt="image" src="https://github.com/user-attachments/assets/d8a19f17-35e3-4d34-a19d-e6dae83bc8b2" />
<img width="536" alt="image" src="https://github.com/user-attachments/assets/78442b5a-3db8-42cb-a8e7-801c4e668b86" />


<h3>🧱 Requirements </h3>

# 🧰 Step 1: Install OCI CLI on Your GCP Ubuntu VM 
Run this command in your GCP SSH terminal:

```
bash -c "$(curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh)"
```

- Press `Enter` for all defaults.
- Wait a few minutes — it installs to $HOME/bin.<br>

After installation, add it to your PATH:
```
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```
Then confirm installation:
```
oci --version
```
- You should see something like 3.x.x. <br>

After the installation finishes:
```
source ~/.bashrc
oci --version
```
# 🧰 Step 2: Configure OCI CLI:

🔑 Step-by-Step: Generate API Key and Set Up OCI CLI

**<h2>✅ 1. Generate Key Pair on Oracle cloud dashboard <br></h2>**

<br>
Go to: <br>
(https://cloud.oracle.com/identity/domains/my-profile/auth-tokens?) <br>
- Click your username.<br>
- Scroll to API Keys → Click "Add API Key" <br>
- Download your private key & public key <br>

**<h2>📋 2. Copy Private Key from downloaded file <br></h2>**

Create the OCI private Key file <br>
Run: [CHANGE `your-username` TO YOUR VM NAME IN BELOW CODE Eg: in my case "/home/hp/.oci/oci_api_key.pem"]

```
nano /home/your-username/.oci/oci_api_key.pem
```

**<h2>📋 3. Create Public Key File <br></h2>**
```
nano ~/.oci/oci_api_key_public.pem
```
- Copy the entire key from downloaded file and paste inside here.
- Press Ctrl + 0 -> save and Enter
- ctrl + x -> Exit

<img width="467" alt="Screenshot 2025-06-29 231439" src="https://github.com/user-attachments/assets/4dee768d-9e5d-42a4-b662-db56475dd3ee" />
<br>

**<h2>📝 4. Create the OCI Config File <br></h2>**
Run:
```
nano ~/.oci/config
```

Paste what you copied from above point (img) (replace placeholders): <br>
Replace: last line with your private key file location - Eg: in my case "key_file=/home/hp/.oci/oci_api_key.pem"
```
**[Below code is for reference, DO NOT use that. It is template and not actual code]**

[DEFAULT]
user=ocid1.user.oc1..aaaaaaaaxxx
fingerprint=xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx
key_file=/home/susanbabu2002/.oci/oci_api_key.pem
tenancy=ocid1.tenancy.oc1..aaaaaaaaxxx
region=us-ashburn-1
Key_file=/home/your-username/.oci/oci_api_key.pem
```
- Then save and exit (Ctrl+O, Enter, Ctrl+X).

**<h2>✅ 5. Test It <br></h2>**
Run:
```
oci iam region list
```
It worked if it shows something like this: 
<br>
<br>
<img width="300" alt="image" src="https://github.com/user-attachments/assets/6d52809b-9fa0-4a53-a693-457a57690f9f" />

# 🧰 Step 3: Gathering Requirements

**<h2>🔍 1. COMPARTMENT_ID: <br></h2>**
Most likely: Use the root compartment (your tenancy) <br>
Run:
```
oci iam compartment list --access-level ACCESSIBLE --compartment-id-in-subtree true
```
You’ll see output like: <br>
<img width="700" alt="image" src="https://github.com/user-attachments/assets/abbef932-951e-4683-b709-1bfbdbed8dc3" />

**<h2>🔍 2. SUBNET_ID: <br></h2>**
You can list subnets in your compartment like this: <br>
Replace with your compartment-id, which we got from previous step <br>
In my case 
> oci network subnet list --compartment-id ocid1.tenancy.oc1..aaaaaaaacbsc......
```
oci network subnet list --compartment-id <replace compartment-id here>
```
You'll output like this: <br>
<img src="https://github.com/user-attachments/assets/f822561c-227f-4cfd-aa0b-a546bff5278d" />

**<h2>🔍 3. IMAGE_ID (Ubuntu 22.04 ARM)</h2>**
- To find your region:
```
  oci iam region-subscription list --query "data[].\"region-name\"" --raw-output
```
- you'll see the output like this: 
<br>
<img src="https://github.com/user-attachments/assets/51e69f92-d5ff-46c3-80ed-a48bf589b0a7" /> <br>
You can find your IMAGE_ID for your `Region` in the below Official link: <br>
[https://docs.oracle.com/en-us/iaas/images/ubuntu-2204/canonical-ubuntu-22-04-2025-05-20-0.htm] <br>

**<h2>4. Known your AVAILABILITY_DOMAINS:</h2>**
<img width="545" alt="image" src="https://github.com/user-attachments/assets/066775d4-4dfb-4e4b-afd0-38f2e04116d7" /> <br>
-Use the code to Known your AVAILABILITY_DOMAINS:
```
oci iam availability-domain list
````

# Step 4: Telegram Notification (This step is completely optional, but useful If you need the notification when VM is created)
<img width="300" alt="image" src="https://github.com/user-attachments/assets/8165bac0-7929-403c-beca-ef2118c0cd4d" />

**<h2>1. Get your Telegram Token:</h2>**
- Search **@BotFather** in your Telegram app <br>
- Press **/start**
- press or type **/newbot**
- Then Name your bot. Eg: createvm_bot
- Then choose username for your bot (username must end with *_bot*) Eg: createvm_bot <br>

<img width="300" src = "https://github.com/user-attachments/assets/ff3a4063-f59b-4623-9cec-845b4a0825c1" /> <br>
- Now you got your Token (Eg. Refer the Img above). keep that and we will be using that in a script
<br>

**<h2>2. Get your Chat ID:</h2>** <br>
- Send a message to your bot, which you have created in previous step. <br>
- Open the URL in your Browser and replace **`<YOUR_BOT_TOKEN>`** with your actual token
``` 
https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates
```
   - Now you got your CHAT_ID
<img width="863" alt="image" src="https://github.com/user-attachments/assets/5659910a-a7b4-42fa-8530-61aed2168847" />


# Step 5: Final Boss - VM Creation Script 
Create an script file:<br>
Run:<br>
```
nano ./launch-arm-vm.sh
```
Paste the script inside and save (ctrl + o -> Enter -> Ctrl + x):<br>
- Replace only these with your actual IDs - **`COMPARTMENT_ID=""`,`SUBNET_ID==""`,`IMAGE_ID=""`,`AVAILABILITY_DOMAINS=("")`,`BOT_TOKEN=""`,`CHAT_ID=""`**
```
#!/bin/bash

# === CONFIGURATION ===
COMPARTMENT_ID=""
SUBNET_ID=""
IMAGE_ID=""  # Ubuntu 22.04 ARM
AVAILABILITY_DOMAINS=("SfSd:US-ASHBURN-AD-1" "SfSd:US-ASHBURN-AD-2" "SfSd:US-ASHBURN-AD-3")
SHAPE="VM.Standard.A1.Flex"
INSTANCE_NAME="free-arm-vm"
SSH_KEY_PATH="$HOME/.ssh/id_rsa.pub"
OCPUS=4
MEMORY=24
BOOT_VOLUME_SIZE=200  # GB

# === TELEGRAM CONFIG ===
BOT_TOKEN=""
CHAT_ID=""

# === SCRIPT START ===
echo "🚀 Starting auto-retry VM launch loop in us-ashburn-1..."

while true; do
  for AD in "${AVAILABILITY_DOMAINS[@]}"; do
    echo "➡️ Trying to launch in $AD..."

    RESPONSE=$(oci compute instance launch \
      --compartment-id "$COMPARTMENT_ID" \
      --availability-domain "$AD" \
      --display-name "$INSTANCE_NAME-$AD" \
      --shape "$SHAPE" \
      --subnet-id "$SUBNET_ID" \
      --image-id "$IMAGE_ID" \
      --metadata "{\"ssh_authorized_keys\": \"$(cat $SSH_KEY_PATH)\"}" \
      --shape-config "{\"ocpus\": $OCPUS, \"memoryInGBs\": $MEMORY}" \
      --boot-volume-size-in-gbs $BOOT_VOLUME_SIZE \
      --wait-for-state RUNNING 2>&1)

    if echo "$RESPONSE" | grep -q "Instance"; then
      echo "✅ VM successfully created in $AD!"
      exit 0
    elif echo "$RESPONSE" | grep -q "Out of host capacity"; then
      echo "❌ No capacity in $AD. Retrying..."
    else
      echo "⚠️ Error encountered:"
      echo "$RESPONSE"
    fi

    sleep 10
  done

  echo "🔁 Retrying all availability domains in 30 seconds..."
  sleep 30
done
```
# Pre-Final Touch <br>
1: Generate SSH Key Pair: <br>
Run this command:
```
ssh-keygen -t rsa -b 2048
```
When prompted:
- File location: just press Enter (default: /home/<name>/.ssh/id_rsa)
- Passphrase: leave empty and press Enter twice

2: Verify: <br>
Run this:
```
cat ~/.ssh/id_rsa.pub
```
- You should now see a long key starting with ssh-rsa AAAA... — this is what your VM will use for SSH access.

# Final Touch <br>

Make It Executable <br>
Run:

```
chmod +x launch-arm-vm.sh
```
Launch the script:
```
./launch-arm-vm.sh
```

# To make the script keep running in background until it get succeed. <br>
⚠️ Run this command so that even if you close your SSH-in-browser, the process keeps running in the background.
Run this:

```
nohup ./launch-arm-vm.sh > output.log 2>&1 &
```

To see the log of the script:
Run:
```
tail -f launch.log
```

