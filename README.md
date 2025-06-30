# Auto-Create-Oracle-Always-Free-ARM-VM-with-Telegram-Notification
Auto-Create Oracle Always Free ARM VM with Telegram Notification

This script automatically retries creating an Oracle Cloud ARM VM in the Always Free tier until capacity is available, then sends a Telegram message upon success.
<hr>
<h3>✅ What This Script Will Do </h3> 
   - Use Ubuntu 22.04 LTS (ARM) image. <br>
    - Try to create an instance of VM.Standard.A1.Flex with: <br>
        - 4 OCPUs <br>
        - 24 GB memory (Always Free limit) <br>
    - Retry in loop if capacity error (out of host capacity). <br>
    - Rotate through ADs: AD-1, AD-2, AD-3. <br>

<hr>

FYI: I used GCP VM to run this script. Why not Oracle itself? If your not going to use all the limit to create this one VM, well you can run the script in Oracle VM itself [Choice is yours] <br>

<h3>
  
✅ GCP VM Setup Quick Guide (Free Tier) <br></h3>
- Go to: https://console.cloud.google.com/compute/instances <br>
- Click: “Create Instance” <br>
- Choose: <br>
- Region: us-west1 or any Free Tier region <br>
- Machine type: e2-micro (Free Tier eligible) <br>
- Boot disk: Ubuntu 22.04 LTS – x86/64, amd64 (recommended) <br>
- Firewall: Allow SSH <br>
- Create the instance <br>


<h3>🧱 Requirements </h3>

# 🧰 Step 1: Install OCI CLI on Your GCP Ubuntu VM 
Run this command in your GCP SSH terminal:


```
bash -c "$(curl -L https://raw.githubusercontent.com/oracle/oci-cli/master/scripts/install/install.sh)"
```
- Press `Enter` for all defaults.
- Wait a few minutes — it installs to $HOME/bin.

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
🔑 Step-by-Step: Generate API Key and Set Up OCI CLI <br>
✅ 1. Generate Key Pair on Oracle cloud dashboard <br>
- Go to: [https://cloud.oracle.com/identity/domains](https://cloud.oracle.com/identity/domains/)
- Click your username.
- Scroll to API Keys → Click "Add API Key"
- Download your private key & public key

📋 2. Copy Private Key from downloaded file <br>
Create the OCI private Key file <br>
Run: [CHANGE `your-username` TO YOUR SYSTEM NAME IN BELOW CODE Eg: in my case "/home/hp/.oci/oci_api_key.pem" ]

```
nano /home/your-username/.oci/oci_api_key.pem
```

📋 3. Create Public Key File<br>
```
nano ~/.oci/oci_api_key_public.pem
```
- Copy the entire key from downloaded file and paste inside here.
- Press Ctrl + 0 -> save and Enter
- ctrl + x -> Exit

<img width="467" alt="Screenshot 2025-06-29 231439" src="https://github.com/user-attachments/assets/4dee768d-9e5d-42a4-b662-db56475dd3ee" />
<br>

📝 5. Create the OCI Config File
Run:
```
nano ~/.oci/config
```

Paste what you copied from above point (img) (replace placeholders): <br>
[Below code is for reference, DO NOT use that. It is template and not actual code] <br>
Replace: last line with your private key file location - Eg: in my case "key_file=/home/hp/.oci/oci_api_key.pem"
```
[DEFAULT]
user=ocid1.user.oc1..aaaaaaaaxxx
fingerprint=xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx
key_file=/home/susanbabu2002/.oci/oci_api_key.pem
tenancy=ocid1.tenancy.oc1..aaaaaaaaxxx
region=us-ashburn-1
Key_file=/home/your-username/.oci/oci_api_key.pem
```
- Then save and exit (Ctrl+O, Enter, Ctrl+X).

✅ 5. Test It
Run:
```
oci iam region list
```
It worked if it shows something like this: <br>
<img width="215" alt="image" src="https://github.com/user-attachments/assets/6d52809b-9fa0-4a53-a693-457a57690f9f" />


