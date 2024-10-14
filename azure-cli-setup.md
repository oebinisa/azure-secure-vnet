# Steps to Set Up Azure CLI  on your Laptop

## Step Overview
1. Install Azure CLI
    - For Windows
    - For MacOS
    - For Linux
2. Verify Installation
3. Log in to Azure
4. Additional Resources

## 1. Install Azure CLI

### For Windows
 - Download the MSI installer from the official Microsoft page: https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-windows
 - Run the installer and follow the instructions to complete the setup

### For MacOS
 - Open your terminal and use Homebrew to install the Azure CLI:

        brew update && brew install azure-cli

### For Linux
 - Install the Azure CLI via your package manager. 
 - For example, for Ubuntu, use:

        sudo apt-get update
        sudo apt-get install -y azure-cli

## 2. Verify Installation
 - The command below will display the current version of Azure CLI installed

        az --version

## 3. Log in to Azure
 - A browser window will open for you to enter your Azure credentials. 
 - Once authenticated, your terminal will show the subscriptions associated with your account

        az login

## 4. Additional Resources:
 - For more Azure CLI commands and details, refer to: https://docs.microsoft.com/en-us/cli/azure/

 End.