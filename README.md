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
- You should see something like 3.x.x.

After the installation finishes:
```
source ~/.bashrc
oci --version
```
# 🧰 Step 2: Configure OCI CLI:
🔑 Step-by-Step: Generate API Key and Set Up OCI CLI
✅ 1. Generate Key Pair on GCP VM
Run this on your GCP VM:
```
mkdir -p ~/.oci
openssl genrsa -out ~/.oci/oci_api_key.pem 2048
openssl rsa -pubout -in ~/.oci/oci_api_key.pem -out ~/.oci/oci_api_key_public.pem
chmod 600 ~/.oci/oci_api_key.pem
```

📋 2. Copy Public Key
Print your public key:
```
cat ~/.oci/oci_api_key_public.pem
```
- Copy the entire key — you'll paste this into your OCI account.

🌐 3. Add API Key to OCI Console
- Go to: https://cloud.oracle.com/identity/users
- Click your username.
- Scroll to API Keys → Click "Add API Key"
- Paste the contents of your oci_api_key_public.pem
<img width="467" alt="Screenshot 2025-06-29 231439" src="https://github.com/user-attachments/assets/4dee768d-9e5d-42a4-b662-db56475dd3ee" />
<br>

📝 4. Create the OCI Config File
Run:
```
nano ~/.oci/config
```

Paste what you copied from above 3rd point (img) (replace placeholders):
[Below code is for reference, do not use that. It is template and not actual code]
```
[DEFAULT]
user=ocid1.user.oc1..aaaaaaaaxxx
fingerprint=xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx
key_file=/home/susanbabu2002/.oci/oci_api_key.pem
tenancy=ocid1.tenancy.oc1..aaaaaaaaxxx
region=us-ashburn-1
```
- Then save and exit (Ctrl+O, Enter, Ctrl+X).

✅ 5. Test It
Run:
```
oci iam region list
```
