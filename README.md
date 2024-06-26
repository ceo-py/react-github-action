
# Deploy React/Next App to VPS via SSH

This GitHub Action automates the deployment of a React/Next application to your VPS (Virtual Private Server) using SSH. Follow the steps below to set up the deployment:


### Step 1: Setup SSH Key on VPS

On your VPS, generate an SSH key pair and configure it to allow GitHub to authenticate:

1. Navigate to the `.ssh` directory if it exists, otherwise create it:
   ```bash
   cd ~/.ssh
   ```

2. Generate an SSH key pair (if not already created):
   ```bash
   ssh-keygen -t rsa -b 4096 -C 'your-email@example.com'
   ```
   Provide a filename when prompted, e.g., `filename`.

3. Copy the public key (`filename.pub`) to your clipboard:
   ```bash
   cat filename.pub | pbcopy  # Use 'cat filename.pub' on Linux
   ```

4. Add the public key to the `authorized_keys` file to allow GitHub to access your VPS:
   ```bash
   cat filename.pub >> authorized_keys
   ```

5. If you encounter permission issues, run the following commands:
   ```bash
   chmod 700 ~/.ssh
   chmod 600 ~/.ssh/filename
   ```

6. Go to the path where your React/Next project will be located on your VPS.

   ```bash
   cd \production\react
   eval "$(ssh-agent -s)"
   ssh-add ~/.ssh/filename
   git clone git@github.com:GITHUB_USERNAME/REPOSITORY_NAME.git
   ```

The directory `repository/name/` will be created automatically after the cloning process completes. Ensure that you use the exact same name as specified below.

### Step 2: Configure GitHub Secrets

Before setting up the GitHub Action, configure the following secrets in your GitHub repository:

- **HOST**: The IP address of your VPS server.
- **USERNAME**: The username used to connect to your VPS (commonly `root`).
- **SSH_PRIVATE_KEY**: The SSH private key generated on your VPS copy the key from **filename**.

To add these secrets, navigate to your GitHub repository > Settings:
```
Security > Secrets and variables and create 3 new secrets.
```
Add each secret as described above.

### Step 3: Configure GitHub Action

Now, configure the GitHub Action to deploy your React application on each push:

1. Create `.github/workflows/deploy.yml` in your repository and add the following configuration:

   ```yaml
        name: Deploy

        # Controls when the action will run. 
        on:
        # Triggers the workflow on push or pull request events but only for the master branch
        push:
            branches: [ main ]

        # A workflow run is made up of one or more jobs that can run sequentially or in parallel
        jobs:
        # This workflow contains a single job called "build"
        build:
            # The type of runner that the job will run on
            runs-on: ubuntu-latest

            # Steps represent a sequence of tasks that will be executed as part of the job
            steps:       
            - name: Deploy using ssh
            uses: appleboy/ssh-action@master
            with:
                host: ${{ secrets.HOST }}
                username: ${{ secrets.USERNAME }}
                key: ${{ secrets.SSH_PRIVATE_KEY }}
                port: 22
                script: |
                eval "$(ssh-agent -s)"
                ssh-add ~/.ssh/filename
                cd /production/react/repository/name/
                git pull origin main
                npm i
                npm run build
   ```

   Replace `repository/name/` with the actual name of your repository on GitHub.

2. Commit and push the changes to your GitHub repository.

### Step 4: Run the GitHub Action

Once the GitHub Action workflow (`deploy.yml`) is set up, it will automatically deploy your React application to the VPS whenever you push changes to the specified branch (e.g., `main`).
