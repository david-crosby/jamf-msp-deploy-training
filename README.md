# Jamf Tenant Building Training
## Rapid Customer Onboarding for MSPs
**Duration:** 2 Hours  
**Target Audience:** MSP Technical Team  
**Focus:** Jamf Pro, Protect & Connect Deployment

---

## Training Objectives

By the end of this session, participants will be able to:
- Rapidly deploy new Jamf Pro tenants for customers
- Implement Jamf Protect and Connect solutions efficiently
- Utilise Jamf Concepts tools to accelerate onboarding
- Automate tenant configuration using CLI and API tools
- Apply MSP-specific best practices for scalable deployments

---

## Session Outline

### Section 1: Introduction & MSP Context (15 minutes)
**Time:** 0:00 - 0:15

#### 1.1 MSP Tenant Building Overview
- Understanding the MSP challenge: managing multiple customer tenants
- Customer onboarding workflow
- Key performance indicators for rapid deployment
- Common pitfalls and how to avoid them

#### 1.2 Jamf Ecosystem for MSPs
- Jamf Pro: Core MDM platform
- Jamf Protect: Endpoint security and threat prevention
- Jamf Connect: Identity and access management
- Integration points between products

#### 1.3 Training Structure
- Hands-on vs conceptual sections
- Lab environment access
- Resources and documentation

---

### Section 2: Jamf Pro Tenant Setup (30 minutes)
**Time:** 0:15 - 0:45

#### 2.1 Initial Tenant Configuration (10 minutes)
**Key Activities:**
- Tenant provisioning and access
- Initial administrator account setup
- Organisation settings configuration
- Apple Business Manager (ABM) integration
- Automated Device Enrolment (ADE) setup

**Configuration Checklist:**
- [ ] Organisation name and branding
- [ ] Time zone and localisation
- [ ] SMTP settings for notifications
- [ ] ABM token upload and verification
- [ ] ADE default prestage configuration
- [ ] Network segments and distribution points

#### 2.2 Jamf Pro Blueprints (10 minutes)
**What are Blueprints?**
Blueprints in Jamf Pro are pre-configured templates that allow you to deploy standardised configurations across devices quickly. They're particularly useful for MSPs managing multiple customers with similar requirements.

**Key Blueprint Components:**
- Configuration profiles
- Policies
- Smart groups
- Extension attributes
- Scripts

**MSP Use Cases:**
- Creating customer-specific blueprints
- Maintaining standardised security baselines
- Rapid deployment for similar industries (e.g., all legal firms, all medical practices)
- Version control for configuration standards

**Practical Application:**
1. Access Blueprints in Jamf Pro: Settings > Computer Management > Blueprints
2. Create a new blueprint or import an existing one
3. Customise for customer-specific requirements
4. Deploy to target devices
5. Monitor compliance through Smart Groups

#### 2.3 API and CLI Setup (10 minutes)
**API Authentication:**
```bash
# Modern approach: Bearer Token Authentication
# Step 1: Create API role and client in Jamf Pro
# Settings > System > API Roles and Clients

# Step 2: Obtain Bearer Token
JAMF_URL="https://customer.jamfcloud.com"
CLIENT_ID="your_client_id"
CLIENT_SECRET="your_client_secret"

# Get token
TOKEN=$(curl -s -X POST "${JAMF_URL}/api/oauth/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "client_id=${CLIENT_ID}&grant_type=client_credentials&client_secret=${CLIENT_SECRET}" \
  | python3 -c "import sys, json; print(json.load(sys.stdin)['access_token'])")

# Use token for API calls
curl -H "Authorization: Bearer ${TOKEN}" \
  "${JAMF_URL}/api/v1/computers-inventory"
```

**Python Automation with python-jamf:**
```python
#!/usr/bin/env python3
"""
Jamf tenant setup automation script
Automates common tenant configuration tasks
"""

import jamf
import os
from typing import Dict, List

def setup_tenant_basics(jamf_url: str, username: str, password: str) -> bool:
    """
    Configure basic tenant settings
    
    Args:
        jamf_url: Jamf Pro server URL
        username: API username
        password: API password
        
    Returns:
        bool: Success status
    """
    try:
        # Connect to Jamf Pro
        jamf_connection = jamf.API(hostname=jamf_url, 
                                    username=username, 
                                    password=password)
        
        # Example: Create smart groups
        smart_groups = [
            {
                "name": "All macOS Devices",
                "criteria": [{"name": "Operating System", "value": "macOS"}]
            },
            {
                "name": "Compliance - FileVault Disabled",
                "criteria": [{"name": "FileVault 2 Status", "value": "Not Encrypted"}]
            }
        ]
        
        # Create groups
        for group in smart_groups:
            # Implementation depends on python-jamf library version
            print(f"Creating smart group: {group['name']}")
        
        return True
        
    except Exception as e:
        print(f"Error: {e}")
        return False

def configure_prestages(jamf_connection, customer_name: str) -> None:
    """
    Configure Automated Device Enrolment prestages
    
    Args:
        jamf_connection: Jamf API connection object
        customer_name: Customer identifier
    """
    # Prestage configuration
    prestage_config = {
        "display_name": f"{customer_name} - Default Prestage",
        "support_phone": "+44 xxx xxxx xxx",
        "support_email": "support@xxxxxxx.example.com",
        "department": customer_name,
        "skip_setup_items": {
            "location": True,
            "privacy": False,  # Show privacy pane
            "biometric": False  # Show Touch ID/Face ID setup
        }
    }
    
    print(f"Configuring prestage for {customer_name}")
    # Implementation continues...

if __name__ == "__main__":
    # Load configuration from environment or config file
    JAMF_URL = os.getenv("JAMF_URL")
    API_USER = os.getenv("JAMF_USER")
    API_PASS = os.getenv("JAMF_PASS")
    
    setup_tenant_basics(JAMF_URL, API_USER, API_PASS)
```

---

### Section 3: Jamf Concepts Tools (40 minutes)
**Time:** 0:45 - 1:25

#### 3.1 Jamf Setup Manager (15 minutes)
**Purpose:** Automate device onboarding during the Setup Assistant phase, before user login.

**Key Features:**
- Runs during Setup Assistant (before user account creation)
- Displays custom branding and progress to end users
- Monitors and installs policies/packages
- Provides visual feedback during device build
- Written in Swift for native macOS performance

**Deployment Architecture:**
```
Apple Device → ADE Enrolment → Jamf Pro Prestage → 
Setup Manager PKG + Profile → Policies Execute → User Desktop
```

**Configuration Steps:**

1. **Download Setup Manager**
   ```bash
   # Available from Jamf GitHub
   # https://github.com/Jamf-Concepts/jamf-setup-manager
   wget https://github.com/Jamf-Concepts/jamf-setup-manager/releases/latest/download/Jamf-Setup-Manager.pkg
   ```

2. **Create Configuration Profile**
   - Navigate to Configuration Profiles in Jamf Pro
   - Create new macOS profile
   - Add Application & Custom Settings payload
   - Preference domain: `com.jamf.setupmanager`
   
   **Example Configuration:**
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
   <plist version="1.0">
   <dict>
       <key>BackgroundImage</key>
       <string>/Library/Application Support/JAMF/Setup Manager/Background.png</string>
       <key>CompanyLogo</key>
       <string>/Library/Application Support/JAMF/Setup Manager/Logo.png</string>
       <key>TitleText</key>
       <string>Welcome to Customer Name</string>
       <key>SubtitleText</key>
       <string>We're configuring your Mac...</string>
       <key>Actions</key>
       <array>
           <dict>
               <key>DisplayName</key>
               <string>Installing Security Software</string>
               <key>Trigger</key>
               <string>install-security</string>
               <key>Type</key>
               <string>policy</string>
           </dict>
           <dict>
               <key>DisplayName</key>
               <string>Configuring Network Settings</string>
               <key>Trigger</key>
               <string>configure-network</string>
               <key>Type</key>
               <string>policy</string>
           </dict>
           <dict>
               <key>DisplayName</key>
               <string>Installing Core Applications</string>
               <key>Trigger</key>
               <string>install-apps</string>
               <key>Type</key>
               <string>policy</string>
           </dict>
       </array>
   </dict>
   </plist>
   ```

3. **Add to Prestage Enrolment**
   - Add Setup Manager PKG to prestage (Position 1)
   - Add Setup Manager configuration profile to prestage (Position 2)
   - Ensure other critical packages follow

4. **Create Policies with Custom Triggers**
   ```bash
   # Policy 1: Install Jamf Protect
   # Trigger: install-security
   # Frequency: Ongoing
   # Execution: At enrolment complete
   
   # Policy 2: Configure network profiles
   # Trigger: configure-network
   
   # Policy 3: Install core apps (Office, Chrome, etc.)
   # Trigger: install-apps
   ```

**MSP Best Practices:**
- Create template Setup Manager configurations
- Store customer logos in standardised locations
- Use consistent trigger naming conventions
- Test on non-production tenant first
- Document customer-specific customisations

**Integration with macOS Onboarding:**
Setup Manager handles pre-login tasks, whilst Jamf's built-in macOS Onboarding (via Self Service) handles post-login tasks like:
- VPP app installation
- User-specific configurations
- Dock customisation
- Wallpaper settings

#### 3.2 Jamf Replicator/Migrator (10 minutes)
**Purpose:** Synchronise configurations between Jamf Pro servers or migrate configurations to new tenants.

**Use Cases for MSPs:**
- Cloning configurations from template tenant to new customer tenant
- Backing up tenant configurations
- Migrating between Jamf Pro instances
- Standardising configurations across multiple customers

**Key Features:**
- Export configurations as XML
- Import to target Jamf Pro server
- Selective export (choose specific objects)
- Batch operations

**Workflow:**

1. **Installation**
   ```bash
   # Download from Jamf resources
   # Available as macOS application
   # Requires Java Runtime Environment
   ```

2. **Export Configuration from Template Tenant**
   - Launch Jamf Replicator
   - Connect to source Jamf Pro server
   - Select objects to export:
     * Configuration Profiles
     * Policies
     * Smart Groups
     * Extension Attributes
     * Scripts
   - Export to local directory

3. **Import to New Customer Tenant**
   - Connect to destination Jamf Pro server
   - Select exported XML files
   - Map dependencies (e.g., distribution points)
   - Customise values (customer name, support contacts)
   - Import selected objects

**Automation Example:**
```bash
#!/bin/zsh
# Script to clone base configuration to new customer tenant
# Uses Jamf API for bulk operations

# Configuration
SOURCE_URL="https://template.jamfcloud.com"
DEST_URL="https://newcustomer.jamfcloud.com"
CONFIG_DIR="/tmp/jamf_configs"

# Export function
export_configs() {
    echo "Exporting configurations from template tenant..."
    
    # Export policies
    curl -X GET "${SOURCE_URL}/JSSResource/policies" \
      -H "Authorization: Bearer ${SOURCE_TOKEN}" \
      -H "Accept: application/xml" \
      -o "${CONFIG_DIR}/policies.xml"
    
    # Export configuration profiles
    curl -X GET "${SOURCE_URL}/JSSResource/osxconfigurationprofiles" \
      -H "Authorization: Bearer ${SOURCE_TOKEN}" \
      -H "Accept: application/xml" \
      -o "${CONFIG_DIR}/profiles.xml"
}

# Import function
import_configs() {
    echo "Importing configurations to new tenant..."
    
    # Parse and import each policy
    # Customise customer-specific values
    # Implementation depends on XML parsing requirements
}

# Execute
mkdir -p ${CONFIG_DIR}
export_configs
import_configs
```

**MSP Considerations:**
- Maintain a "golden tenant" with base configurations
- Version control your exported configurations
- Document customisation requirements per customer
- Test imports on staging environment first
- Be aware of object dependencies (scripts referenced in policies, etc.)

#### 3.3 Jamf Compliance Editor (10 minutes)
**Purpose:** Create and manage security compliance baselines based on industry standards (CIS, NIST, DISA STIG).

**Why It Matters for MSPs:**
- Customers increasingly require compliance certifications
- Automated policy creation saves hours of manual work
- Standardised approach across multiple customers
- Built-in remediation scripts

**Supported Frameworks:**
- CIS Benchmarks (Level 1, Level 2)
- NIST 800-53
- NIST 800-171
- DISA STIG
- CMMC
- Custom baselines

**Workflow:**

1. **Installation**
   - Download from: https://github.com/Jamf-Concepts/jamf-compliance-editor
   - macOS-only application (runs on admin's Mac)
   - No installation required on managed devices

2. **Create Compliance Project**
   - Launch Jamf Compliance Editor
   - Create New Project
   - Select macOS version (Sonoma, Sequoia, etc.)
   - Choose compliance framework (e.g., CIS Level 1)
   - Save project locally

3. **Customise Baseline**
   - Review each control
   - Enable/disable controls based on customer requirements
   - Modify Organisation Defined Values (ODVs)
   - Add custom rules if needed

4. **Generate Jamf Pro Objects**
   - Click "Generate"
   - Application creates:
     * Configuration profiles for each security setting
     * Extension attributes for compliance reporting
     * Scripts for remediation
     * Smart groups for non-compliant devices

5. **Upload to Jamf Pro**
   - Connect to customer's Jamf Pro tenant
   - Select objects to upload
   - Review and confirm
   - Upload profiles, scripts, and extension attributes

**Example: CIS Benchmark Controls**
```
Control 2.1.1: Turn off Bluetooth when not in use
- Configuration Profile: Restricts Bluetooth
- Extension Attribute: Reports Bluetooth status
- Smart Group: Devices with Bluetooth enabled
- Remediation: Policy to disable Bluetooth

Control 2.5.1: Enable FileVault
- Configuration Profile: Deploys FileVault with recovery key escrow
- Extension Attribute: Reports encryption status
- Smart Group: Unencrypted devices
- Remediation: Policy to enable FileVault
```

**MSP Workflow:**
```
Customer Onboarding → Compliance Discussion → 
Select Framework → Customise in Editor → 
Generate Profiles → Upload to Tenant → 
Deploy to Devices → Monitor Compliance
```

**Reporting:**
- Use extension attributes to collect compliance data
- Create smart groups for non-compliant devices
- Generate reports for customer compliance reviews
- Schedule regular compliance scans

**Best Practices:**
- Create project templates for each compliance framework
- Version control your compliance projects
- Document customer-specific deviations from standards
- Test compliance profiles on test devices first
- Schedule regular compliance reviews with customers

#### 3.4 Scripting and Automation (5 minutes)
**Purpose:** Custom automation for repetitive tasks and integration with other systems.

**Common Automation Scenarios:**
- Bulk device attribute updates
- Custom reporting
- Integration with ticketing systems
- Automated policy deployment
- Backup and disaster recovery

**ZSH Script Example: Bulk Enrolment Invitation**
```zsh
#!/bin/zsh
# Script to send enrolment invitations to new users
# Author: IT Team
# Usage: ./send_invitations.sh customer_name user_list.csv

CUSTOMER=$1
USER_FILE=$2
JAMF_URL="https://${CUSTOMER}.jamfcloud.com"

# Validate inputs
if [[ -z "$CUSTOMER" ]] || [[ ! -f "$USER_FILE" ]]; then
    echo "Usage: $0 customer_name user_list.csv"
    exit 1
fi

# Function: Get Bearer token
get_token() {
    local client_id=$1
    local client_secret=$2
    
    curl -s -X POST "${JAMF_URL}/api/oauth/token" \
      -H "Content-Type: application/x-www-form-urlencoded" \
      -d "client_id=${client_id}&grant_type=client_credentials&client_secret=${client_secret}" \
      | python3 -c "import sys, json; print(json.load(sys.stdin)['access_token'])"
}

# Function: Send enrolment invitation
send_invitation() {
    local email=$1
    local token=$2
    
    curl -X POST "${JAMF_URL}/api/v1/device-enrollments/public-key/invitations" \
      -H "Authorization: Bearer ${token}" \
      -H "Content-Type: application/json" \
      -d "{\"invitation\":{\"invitationEmail\":\"${email}\"}}"
}

# Main execution
TOKEN=$(get_token "$CLIENT_ID" "$CLIENT_SECRET")

# Read CSV file and send invitations
while IFS=, read -r name email department; do
    echo "Sending invitation to: $email"
    send_invitation "$email" "$TOKEN"
    sleep 2  # Rate limiting
done < "$USER_FILE"

echo "Invitations sent successfully"
```

**Python Script Example: Configuration Backup**
```python
#!/usr/bin/env python3
"""
Backup Jamf Pro tenant configuration
Exports all major configuration objects to JSON files
"""

import requests
import json
import os
from datetime import datetime
from pathlib import Path

class JamfBackup:
    """Handle Jamf Pro configuration backups"""
    
    def __init__(self, jamf_url: str, client_id: str, client_secret: str):
        """
        Initialise backup handler
        
        Args:
            jamf_url: Jamf Pro server URL
            client_id: API client ID
            client_secret: API client secret
        """
        self.jamf_url = jamf_url.rstrip('/')
        self.client_id = client_id
        self.client_secret = client_secret
        self.token = None
        
    def get_token(self) -> str:
        """Obtain Bearer token from Jamf Pro"""
        url = f"{self.jamf_url}/api/oauth/token"
        data = {
            "client_id": self.client_id,
            "grant_type": "client_credentials",
            "client_secret": self.client_secret
        }
        
        response = requests.post(url, data=data)
        response.raise_for_status()
        
        self.token = response.json()['access_token']
        return self.token
    
    def backup_policies(self, output_dir: Path) -> None:
        """
        Backup all policies
        
        Args:
            output_dir: Directory to save backup files
        """
        headers = {"Authorization": f"Bearer {self.token}"}
        
        # Get all policy IDs
        url = f"{self.jamf_url}/JSSResource/policies"
        response = requests.get(url, headers=headers)
        policies = response.json()['policies']
        
        # Backup each policy
        policy_dir = output_dir / "policies"
        policy_dir.mkdir(exist_ok=True)
        
        for policy in policies:
            policy_id = policy['id']
            policy_name = policy['name']
            
            # Get full policy details
            url = f"{self.jamf_url}/JSSResource/policies/id/{policy_id}"
            response = requests.get(url, headers=headers)
            
            # Save to file
            filename = policy_dir / f"{policy_id}_{policy_name}.json"
            with open(filename, 'w') as f:
                json.dump(response.json(), f, indent=2)
            
            print(f"Backed up policy: {policy_name}")
    
    def backup_profiles(self, output_dir: Path) -> None:
        """Backup all configuration profiles"""
        headers = {"Authorization": f"Bearer {self.token}"}
        
        # Get all profile IDs
        url = f"{self.jamf_url}/JSSResource/osxconfigurationprofiles"
        response = requests.get(url, headers=headers)
        profiles = response.json()['os_x_configuration_profiles']
        
        # Backup each profile
        profile_dir = output_dir / "profiles"
        profile_dir.mkdir(exist_ok=True)
        
        for profile in profiles:
            profile_id = profile['id']
            profile_name = profile['name']
            
            # Get full profile details
            url = f"{self.jamf_url}/JSSResource/osxconfigurationprofiles/id/{profile_id}"
            response = requests.get(url, headers=headers)
            
            # Save to file
            filename = profile_dir / f"{profile_id}_{profile_name}.json"
            with open(filename, 'w') as f:
                json.dump(response.json(), f, indent=2)
            
            print(f"Backed up profile: {profile_name}")
    
    def run_backup(self, customer_name: str) -> None:
        """
        Execute complete backup
        
        Args:
            customer_name: Customer identifier for backup directory
        """
        # Create backup directory structure
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        backup_dir = Path(f"backups/{customer_name}_{timestamp}")
        backup_dir.mkdir(parents=True, exist_ok=True)
        
        print(f"Starting backup for {customer_name}")
        print(f"Backup directory: {backup_dir}")
        
        # Obtain authentication token
        self.get_token()
        
        # Perform backups
        self.backup_policies(backup_dir)
        self.backup_profiles(backup_dir)
        # Add more backup functions as needed
        
        print(f"Backup completed: {backup_dir}")

# Usage
if __name__ == "__main__":
    # Load credentials from environment
    JAMF_URL = os.getenv("JAMF_URL")
    CLIENT_ID = os.getenv("JAMF_CLIENT_ID")
    CLIENT_SECRET = os.getenv("JAMF_CLIENT_SECRET")
    CUSTOMER = os.getenv("CUSTOMER_NAME", "customer")
    
    backup = JamfBackup(JAMF_URL, CLIENT_ID, CLIENT_SECRET)
    backup.run_backup(CUSTOMER)
```

---

### Section 4: Jamf Protect Deployment (20 minutes)
**Time:** 1:25 - 1:45

#### 4.1 Jamf Protect Overview (5 minutes)
**What is Jamf Protect?**
- Native macOS endpoint security solution
- Real-time threat prevention and detection
- Integration with Jamf Pro for unified management
- Built on macOS System Extensions framework

**Key Features:**
- Malware prevention and detection
- USB device control
- Network traffic filtering
- Application behaviour monitoring
- Threat intelligence integration
- Compliance monitoring

**Architecture:**
```
macOS Device → Jamf Protect Agent → 
Jamf Protect Cloud → Analytics & Alerting → 
MSP SOC Dashboard
```

#### 4.2 Tenant Configuration (7 minutes)
**Initial Setup:**

1. **Access Jamf Protect Portal**
   - Login to Jamf Protect cloud console
   - Navigate to Settings → Tenants
   - Configure tenant details

2. **Create Plans**
   Plans define the security configuration for devices.
   
   **Standard MSP Plan Structure:**
   - **Plan A: Standard Security** (Most customers)
     * Anti-malware enabled
     * Basic USB controls
     * Standard threat prevention
   
   - **Plan B: Enhanced Security** (Regulated industries)
     * All Plan A features
     * Strict USB controls
     * Network content filtering
     * Custom threat hunting rules
   
   - **Plan C: Maximum Security** (High-security environments)
     * All Plan B features
     * Application whitelisting
     * Advanced behavioural analysis
     * Real-time alerting to SOC

3. **Configure Plan Settings**
   ```
   Navigate to: Plans → Create New Plan
   
   Settings to Configure:
   - Plan Name: e.g., "Xxxxxx-Customer-Standard"
   - Anti-Malware: Enabled
   - Unified Logs: Enabled (important for troubleshooting)
   - Analytics: Enabled
   - Auto-Update: Enabled
   - Custom Rules: Import from library or create
   ```

4. **Configure Alerts**
   - Set up alert destinations (email, webhook, SIEM)
   - Define alert thresholds
   - Configure incident response workflows

5. **USB Device Control**
   ```
   Navigate to: Plans → [Your Plan] → Threat Prevention → 
                 USB Storage & Network Drives
   
   Options:
   - Allow all USB devices
   - Block all USB devices
   - Block unauthorised USB devices
   - Require authorisation for USB devices
   ```

#### 4.3 Deployment via Jamf Pro (8 minutes)
**Integration Steps:**

1. **Link Jamf Protect to Jamf Pro**
   ```
   In Jamf Protect:
   Settings → Jamf Pro Integration → Add Server
   - Enter Jamf Pro URL
   - Provide API credentials
   - Test connection
   
   In Jamf Pro:
   Settings → Jamf Applications → Jamf Protect
   - Verify integration status
   ```

2. **Deploy Jamf Protect Agent**
   
   **Method 1: Via Jamf Pro (Recommended)**
   ```
   After integration, Jamf Protect installer and plans are 
   automatically available in Jamf Pro.
   
   Create Policy:
   - Name: "Install Jamf Protect"
   - Trigger: Enrolment Complete
   - Frequency: Once per computer
   - Package: Jamf Protect installer (auto-populated)
   - Scope: All computers or specific smart group
   ```

3. **Deploy Configuration Profile (Plan)**
   ```
   Jamf Pro → Configuration Profiles → New
   - General: Name = "Jamf Protect - Standard Plan"
   - Jamf Protect payload: Select plan from dropdown
   - Scope: Same as installer policy
   - Distribution: Make Available in Self Service = No
   ```

4. **Verify Deployment**
   ```bash
   # On managed device, check if agent is running
   ps aux | grep JamfProtect
   
   # Check for configuration profile
   profiles -P | grep com.jamf.protect
   
   # Verify cloud connectivity
   sudo jamf_protect diagnostics --check-connectivity
   ```

**Deployment Sequence:**
```
1. Deploy configuration profile (Plan) first
2. Deploy Jamf Protect installer package
3. Agent installs and loads System Extensions
4. Agent connects to Jamf Protect cloud
5. Security policies active
```

**MSP Considerations:**
- Create plan templates for different security levels
- Use Smart Groups to target specific plans to device groups
- Monitor deployment status via Jamf Protect dashboard
- Set up customer-specific alerting rules
- Document customer security requirements

**Troubleshooting Common Issues:**
```bash
# Issue: System Extension not approved
# Solution: Deploy PPPC profile before Protect installer

# Issue: Agent not checking in
# Check network connectivity
sudo jamf_protect diagnostics --check-connectivity

# Issue: Plan not applied
# Verify configuration profile installed
profiles -P

# Force plan refresh
sudo jamf_protect refresh-plan
```

---

### Section 5: Jamf Connect Deployment (15 minutes)
**Time:** 1:45 - 2:00

#### 5.1 Jamf Connect Overview (3 minutes)
**What is Jamf Connect?**
Cloud identity integration for macOS, enabling:
- SSO-based local account creation
- Password synchronisation between cloud and local accounts
- Conditional access integration
- Self-service password management

**Key Components:**
- **Jamf Connect Login:** Replaces macOS login window
- **Jamf Connect Menu Bar App:** Post-login password sync and admin escalation
- **Jamf Connect Configuration:** IdP integration settings

**Supported Identity Providers:**
- Azure AD / Microsoft Entra ID
- Okta
- Google Workspace
- Ping Identity
- Generic OIDC providers

#### 5.2 Azure AD / Entra ID Configuration (6 minutes)
**Prerequisites:**
- Azure AD tenant admin access
- App registration permissions
- Understanding of OAuth 2.0 / OpenID Connect

**Step-by-Step Configuration:**

1. **Create App Registration in Azure AD**
   ```
   Azure Portal → Azure Active Directory → App registrations → New registration
   
   Settings:
   - Name: "Jamf Connect - Customer Name"
   - Supported account types: Single tenant
   - Redirect URI: https://127.0.0.1/jamfconnect
   
   After creation:
   - Note the Application (client) ID
   - Note the Directory (tenant) ID
   ```

2. **Configure API Permissions**
   ```
   App Registration → API Permissions → Add permission
   
   Required Microsoft Graph Permissions:
   - User.Read (Delegated) - Read user profile
   - offline_access (Delegated) - Maintain access
   - openid (Delegated) - OpenID Connect
   - profile (Delegated) - User profile information
   - email (Delegated) - Email address
   
   Click "Grant admin consent"
   ```

3. **Create Client Secret**
   ```
   App Registration → Certificates & secrets → New client secret
   
   - Description: "Jamf Connect Secret"
   - Expires: 24 months (maximum)
   - Copy secret value immediately (won't be shown again)
   ```

4. **Configure Token Settings**
   ```
   App Registration → Token configuration → Add optional claim
   
   Token type: ID
   Claims to add:
   - email
   - family_name
   - given_name
   - upn (User Principal Name)
   ```

#### 5.3 Jamf Connect Deployment (6 minutes)
**Configuration Profile Creation:**

1. **Download Jamf Connect**
   ```
   From Jamf Connect downloads page:
   - Jamf Connect installer package
   - Jamf Connect Configuration utility (for profile creation)
   ```

2. **Create Jamf Connect Login Profile**
   
   Use Jamf Connect Configuration utility:
   ```
   Launch Jamf Connect Configuration.app
   
   Authentication Tab:
   - Provider: Azure
   - Tenant ID: [Your Azure AD Tenant ID]
   - Client ID: [Your App Registration Client ID]
   - Redirect URI: https://127.0.0.1/jamfconnect
   
   User Experience Tab:
   - Show Welcome Text: Enable
   - Welcome Title: "Welcome to [Customer Name]"
   - Logo: Upload customer logo
   - Background Image: Upload custom background
   
   Password Sync Tab:
   - Enable Password Sync: Yes
   - Require Password Confirmation: Yes
   
   Advanced Tab:
   - Create Admin User: No (or Yes if required)
   - Migrate Users: No (for new deployments)
   - Hide Get Started Experience: Yes
   
   Save profile as: JamfConnect-Login.mobileconfig
   ```

3. **Create Jamf Connect Menu Bar App Profile**
   ```
   Jamf Connect Configuration.app → Menu Bar App
   
   Settings:
   - Enable Menu Bar App: Yes
   - Password Sync: Enabled
   - Notify User on Password Change: Yes
   - Admin Privileges: Configure as needed
   
   Self Service Actions:
   - Reset Password: Enabled
   - Temporary Admin Rights: Enabled (10 minutes default)
   - Lock Screen: Enabled
   
   Save profile as: JamfConnect-MenuBar.mobileconfig
   ```

4. **Upload to Jamf Pro**
   ```
   Jamf Pro → Configuration Profiles → Upload
   
   Profile 1: Jamf Connect Login
   - Upload JamfConnect-Login.mobileconfig
   - Scope: All computers or prestage enrolment
   - Level: Computer Level
   
   Profile 2: Jamf Connect Menu Bar
   - Upload JamfConnect-MenuBar.mobileconfig
   - Scope: All computers
   - Level: Computer Level
   ```

5. **Deploy Jamf Connect Package**
   ```
   Jamf Pro → Policies → New
   
   Policy: "Install Jamf Connect"
   - Package: JamfConnect.pkg
   - Trigger: Enrolment Complete
   - Execution Frequency: Once per computer
   - Scope: All computers
   - Add to prestage if zero-touch deployment
   ```

**Deployment Order for Prestage:**
```
Position 1: Jamf Connect configuration profiles
Position 2: Jamf Connect installer package
Position 3: Other packages/profiles
```

**Testing Deployment:**
```
1. Wipe a test Mac
2. Connect to internet
3. Power on and go through Setup Assistant
4. At Remote Management screen, proceed
5. Jamf Connect login window should appear
6. Sign in with Azure AD credentials
7. Local account created with cloud identity
8. Verify password sync works
9. Test menu bar app functionality
```

**MSP Best Practices:**
- Create template configurations per IdP
- Store app registrations securely (password manager)
- Document client secrets and expiry dates
- Test with pilot users before full deployment
- Provide end-user documentation for password sync
- Configure fallback authentication method

---

## Section 6: MSP Workflows and Best Practices (15 minutes)
**Time:** 2:00 - 2:15

### 6.1 Customer Onboarding Checklist

**Pre-Onboarding Phase:**
- [ ] Customer requirements gathering
  - Device count and types
  - Security compliance requirements
  - Identity provider details
  - Software requirements
  - Existing infrastructure
- [ ] Jamf tenant provisioning
- [ ] Apple Business Manager setup
- [ ] Identity provider app registration
- [ ] Network and firewall configuration

**Initial Configuration Phase:**
- [ ] Tenant branding and settings
- [ ] Administrator accounts created
- [ ] API roles and clients configured
- [ ] ABM token uploaded
- [ ] DEP profile created
- [ ] Distribution point configured
- [ ] Notification settings

**Security Phase:**
- [ ] Jamf Protect tenant configured
- [ ] Security plans created
- [ ] Jamf Connect configured
- [ ] Compliance baseline selected
- [ ] Configuration profiles deployed
- [ ] FileVault configuration

**Application Phase:**
- [ ] VPP token uploaded
- [ ] Core applications added
- [ ] Self Service customised
- [ ] macOS Onboarding configured
- [ ] Setup Manager configured (if applicable)

**Policy Phase:**
- [ ] Smart groups created
- [ ] Policies configured
- [ ] Update management configured
- [ ] Reporting configured
- [ ] Backup procedures established

**Testing Phase:**
- [ ] Test device enrolment
- [ ] Verify all policies apply correctly
- [ ] Test user experience
- [ ] Validate security controls
- [ ] Performance testing

**Handover Phase:**
- [ ] Documentation provided to customer
- [ ] Training delivered
- [ ] Support procedures established
- [ ] Monitoring configured
- [ ] Success metrics defined

### 6.2 Template Library Management

**Maintaining Configuration Templates:**
```
Template Structure:
/Templates
  /Jamf-Pro
    /Policies
      - Base-Security-Policy.xml
      - Software-Update-Policy.xml
      - Backup-Policy.xml
    /Profiles
      - FileVault-Profile.mobileconfig
      - WiFi-Profile.mobileconfig
      - Restrictions-Profile.mobileconfig
    /Smart-Groups
      - OS-Version-Groups.xml
      - Compliance-Groups.xml
  /Jamf-Protect
    /Plans
      - Standard-Security-Plan.json
      - Enhanced-Security-Plan.json
  /Jamf-Connect
    /Azure-AD
      - Login-Profile-Template.mobileconfig
      - MenuBar-Profile-Template.mobileconfig
    /Okta
      - Login-Profile-Template.mobileconfig
  /Scripts
    - tenant-setup.py
    - backup-config.py
    - bulk-invite.zsh
```

**Version Control:**
```bash
# Initialise Git repository for templates
cd /path/to/templates
git init
git add .
git commit -m "feat: Initial template library"

# Track changes
git add Jamf-Pro/Policies/Base-Security-Policy.xml
git commit -m "feat(policy): Update FileVault requirements to comply with CIS 2.5"

# Use Conventional Commits format:
# feat: New feature
# fix: Bug fix
# docs: Documentation
# chore: Maintenance
# refactor: Code restructuring
```

### 6.3 Automation Workflows

**Automated Tenant Provisioning:**
```python
#!/usr/bin/env python3
"""
Automated Jamf tenant provisioning for MSP
Orchestrates complete tenant setup from template
"""

import os
import json
from pathlib import Path
from typing import Dict, List
import requests

class TenantProvisioning:
    """Automate Jamf tenant setup for new customers"""
    
    def __init__(self, config_file: str):
        """
        Initialise provisioning with customer configuration
        
        Args:
            config_file: Path to customer configuration JSON
        """
        with open(config_file, 'r') as f:
            self.config = json.load(f)
        
        self.jamf_url = self.config['jamf_url']
        self.customer_name = self.config['customer_name']
        self.template_dir = Path(self.config['template_directory'])
    
    def setup_organisation_settings(self):
        """Configure basic organisation settings"""
        print(f"Configuring organisation settings for {self.customer_name}")
        
        settings = {
            "name": self.customer_name,
            "address": self.config['address'],
            "phone": self.config['phone'],
            "email": self.config['support_email']
        }
        
        # API call to update settings
        # Implementation continues...
    
    def create_smart_groups(self):
        """Create standard smart groups from templates"""
        print("Creating smart groups...")
        
        groups_dir = self.template_dir / "Jamf-Pro" / "Smart-Groups"
        
        for group_file in groups_dir.glob("*.xml"):
            print(f"Creating: {group_file.stem}")
            # Load template
            # Customise for customer
            # Upload via API
    
    def deploy_security_baseline(self):
        """Deploy security configuration profiles"""
        print("Deploying security baseline...")
        
        profiles_dir = self.template_dir / "Jamf-Pro" / "Profiles"
        
        # Deploy each profile
        # Scope appropriately
        # Verify deployment
    
    def configure_jamf_protect(self):
        """Set up Jamf Protect integration"""
        print("Configuring Jamf Protect...")
        
        # Create plan from template
        # Customise for customer security level
        # Deploy to devices
    
    def configure_jamf_connect(self):
        """Set up Jamf Connect with customer IdP"""
        print("Configuring Jamf Connect...")
        
        idp = self.config['identity_provider']
        
        if idp == 'azure':
            self.setup_azure_connect()
        elif idp == 'okta':
            self.setup_okta_connect()
    
    def run_full_provisioning(self):
        """Execute complete tenant provisioning"""
        steps = [
            ("Organisation Settings", self.setup_organisation_settings),
            ("Smart Groups", self.create_smart_groups),
            ("Security Baseline", self.deploy_security_baseline),
            ("Jamf Protect", self.configure_jamf_protect),
            ("Jamf Connect", self.configure_jamf_connect)
        ]
        
        print(f"\n=== Starting Provisioning for {self.customer_name} ===\n")
        
        for step_name, step_func in steps:
            try:
                print(f"\n--- {step_name} ---")
                step_func()
                print(f"✓ {step_name} completed")
            except Exception as e:
                print(f"✗ {step_name} failed: {e}")
                raise
        
        print(f"\n=== Provisioning Complete ===")
        print(f"Tenant URL: {self.jamf_url}")
        print(f"Customer: {self.customer_name}")

# Usage
if __name__ == "__main__":
    config_file = "customer-config.json"
    
    provisioning = TenantProvisioning(config_file)
    provisioning.run_full_provisioning()
```

**Customer Configuration File Example:**
```json
{
  "customer_name": "Acme Corporation",
  "jamf_url": "https://acme.jamfcloud.com",
  "template_directory": "/path/to/templates",
  "identity_provider": "azure",
  "azure_tenant_id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "address": "123 High Street, London",
  "phone": "+44 20 xxxx xxxx",
  "support_email": "support@xxxxxx.example.com",
  "security_level": "enhanced",
  "compliance_framework": "CIS Level 1",
  "device_count": 50,
  "vpp_token": "/path/to/vpp/token",
  "abm_token": "/path/to/abm/token"
}
```

### 6.4 Monitoring and Maintenance

**Key Metrics to Monitor:**
- Enrolment success rate
- Policy compliance percentage
- Software update compliance
- Security alert volume
- Device health scores
- User support tickets

**Automated Health Checks:**
```bash
#!/bin/zsh
# Daily health check script for customer tenants
# Runs via cron: 0 8 * * * /path/to/health-check.sh

# Configuration
CUSTOMERS=("customer1" "customer2" "customer3")
REPORT_EMAIL="alerts@xxxxxx.example.com"

# Function: Check tenant health
check_tenant() {
    local customer=$1
    local url="https://${customer}.jamfcloud.com"
    
    echo "Checking ${customer}..."
    
    # Check API accessibility
    http_code=$(curl -s -o /dev/null -w "%{http_code}" "${url}/healthCheck.html")
    
    if [[ $http_code -eq 200 ]]; then
        echo "✓ ${customer}: Healthy"
    else
        echo "✗ ${customer}: Unhealthy (HTTP ${http_code})"
        # Send alert
    fi
    
    # Check enrolment count (last 24 hours)
    # Check policy failures
    # Check certificate expiry
    # Generate report
}

# Execute checks
for customer in "${CUSTOMERS[@]}"; do
    check_tenant "$customer"
done
```

### 6.5 Documentation Standards

**Customer Documentation Package:**
1. **Tenant Overview Document**
   - Tenant URL and access credentials
   - Administrator contacts
   - Support procedures
   - Escalation paths

2. **Configuration Summary**
   - Deployed policies
   - Security settings
   - Application catalogue
   - User groups and scoping

3. **End-User Guides**
   - Device enrolment instructions
   - Self Service usage
   - Password sync procedures
   - Requesting administrative access

4. **Admin Procedures**
   - Adding new users
   - Device decommissioning
   - Software deployment
   - Report generation
   - Backup and recovery

5. **Runbook**
   - Common troubleshooting procedures
   - Emergency contacts
   - Maintenance schedules
   - Change management process

---

## Section 7: Q&A and Practical Exercise (15 minutes)
**Time:** 2:15 - 2:30

### Practical Exercise: Mini Tenant Build
**Scenario:** New customer "TechStart Ltd" with 25 MacBook Airs

**Requirements:**
- Basic security configuration
- Jamf Protect with standard plan
- Azure AD integration with Jamf Connect
- Core applications: Chrome, Slack, Zoom

**Task:** Plan the deployment steps and identify which tools would be used

**Expected Workflow:**
1. Jamf Pro tenant provisioned
2. ABM token configured
3. Use Jamf Compliance Editor for CIS Level 1 baseline
4. Configure Jamf Connect for Azure AD
5. Deploy Jamf Protect standard plan
6. Use Setup Manager for device onboarding
7. Configure policies for core applications
8. Test with pilot device

### Q&A Topics to Cover
- Integration challenges
- Troubleshooting common issues
- Scaling considerations
- Customer communication strategies
- Tool selection criteria

---

## Appendix A: Quick Reference Commands

### Jamf Pro API
```bash
# Get Bearer Token
curl -X POST "${JAMF_URL}/api/oauth/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "client_id=${CLIENT_ID}&grant_type=client_credentials&client_secret=${CLIENT_SECRET}"

# Get all computers
curl -H "Authorization: Bearer ${TOKEN}" \
  "${JAMF_URL}/api/v1/computers-inventory"

# Get computer by ID
curl -H "Authorization: Bearer ${TOKEN}" \
  "${JAMF_URL}/api/v1/computers-inventory/${COMPUTER_ID}"

# Get all policies
curl -H "Authorization: Bearer ${TOKEN}" \
  "${JAMF_URL}/JSSResource/policies"

# Get configuration profiles
curl -H "Authorization: Bearer ${TOKEN}" \
  "${JAMF_URL}/JSSResource/osxconfigurationprofiles"
```

### Jamf Protect Agent Commands
```bash
# Check agent status
sudo jamf_protect status

# Test cloud connectivity
sudo jamf_protect diagnostics --check-connectivity

# Refresh plan
sudo jamf_protect refresh-plan

# View current plan
sudo jamf_protect current-plan

# Generate diagnostic bundle
sudo jamf_protect diagnostics --generate-bundle
```

### Jamf Connect Commands
```bash
# Check Jamf Connect status
sudo log show --predicate 'process == "JamfConnect"' --last 1h

# Force password sync
/usr/local/bin/jamfConnect --sync

# Reset Jamf Connect
sudo rm -rf /Library/Application\ Support/JamfConnect/
sudo rm -rf /Library/LaunchDaemons/com.jamf.connect.*
```

### Useful macOS Commands
```bash
# Check MDM enrolment
sudo profiles status

# List installed profiles
sudo profiles -P

# Check FileVault status
sudo fdesetup status

# Check Gatekeeper status
spctl --status

# Check System Extensions
systemextensionsctl list
```

---

## Appendix B: Resources and Links

### Jamf Resources
- Jamf Pro Documentation: https://docs.jamf.com/
- Jamf Protect Documentation: https://docs.jamf.com/jamf-protect/
- Jamf Connect Documentation: https://docs.jamf.com/jamf-connect/
- Jamf Developer Portal: https://developer.jamf.com/
- Jamf Nation Community: https://community.jamf.com/
- Jamf Training: https://www.jamf.com/training/

### Jamf Concepts
- Jamf Concepts GitHub: https://github.com/Jamf-Concepts/
- Setup Manager: https://github.com/Jamf-Concepts/jamf-setup-manager
- Compliance Editor: https://github.com/Jamf-Concepts/jamf-compliance-editor
- Jamf Concepts Site: https://concepts.jamf.com/

### API and Automation
- python-jamf: https://github.com/univ-of-utah-marriott-library-apple/python-jamf
- jctl: https://github.com/univ-of-utah-marriott-library-apple/jctl
- Jamf Pro SDK for Python: https://github.com/Jamf-Custom-Profile-Schemas

### Compliance Resources
- macOS Security Compliance Project: https://github.com/usnistgov/macos_security
- CIS Benchmarks: https://www.cisecurity.org/cis-benchmarks/
- NIST 800-53: https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final

### Apple Resources
- Apple Business Manager: https://business.apple.com/
- Apple Developer Documentation: https://developer.apple.com/documentation/
- Apple Platform Deployment: https://support.apple.com/en-gb/guide/deployment/welcome/web

### Community Resources
- MacAdmins Slack: https://www.macadmins.org/
- r/macsysadmin: https://reddit.com/r/macsysadmin
- Der Flounder Blog: https://derflounder.wordpress.com/

---

## Appendix C: Troubleshooting Guide

### Common Issues and Solutions

#### Jamf Pro
**Issue:** Devices not enrolling
- Check ABM token status and expiry
- Verify network connectivity
- Check prestage assignment
- Review enrolment logs

**Issue:** Policies not running
- Verify scope (smart groups, all computers)
- Check trigger configuration
- Review policy logs on device
- Ensure device has checked in recently

**Issue:** Configuration profiles not installing
- Check profile scope
- Verify no conflicts with existing profiles
- Review MDM protocol logs
- Check for certificate issues

#### Jamf Protect
**Issue:** Agent not checking in
- Verify network connectivity to Jamf Protect cloud
- Check plan configuration profile installed
- Review System Extension approval status
- Check firewall rules

**Issue:** Threats not being blocked
- Verify plan settings
- Check Prevention mode vs Audit mode
- Review custom rules
- Ensure agent is up to date

**Issue:** High false positive rate
- Tune analytic rules
- Add exclusions for known-good apps
- Review machine learning thresholds
- Consider switching plans

#### Jamf Connect
**Issue:** Login window not appearing
- Verify configuration profile scope
- Check LaunchAgent/LaunchDaemon loaded
- Review system logs
- Confirm IdP app registration correct

**Issue:** Password sync failing
- Check network connectivity to IdP
- Verify user credentials
- Review menu bar app logs
- Confirm token hasn't expired

**Issue:** Cannot create local account
- Check IdP claims configuration
- Verify username format settings
- Review SecureToken requirements
- Check for existing account conflicts

---

## Training Completion

Congratulations on completing the Jamf Tenant Building training!

### Next Steps
1. Review the training materials and notes
2. Practice in a lab environment
3. Build your first customer tenant using the checklist
4. Document any issues or improvements
5. Share knowledge with the wider team

### Feedback
Please provide feedback on this training session to help us improve future sessions.


