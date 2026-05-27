# ──────────────────────────────────────────────────────────────
# Production Grade Jenkins Dockerfile
# Includes: Docker, kubectl, AWS CLI, Terraform, 
#           SonarQube Scanner, Trivy, Git, SSH, Maven, Node.js
# ──────────────────────────────────────────────────────────────

FROM jenkins/jenkins:lts

# Switch to root to install packages
USER root

# ── System Dependencies ────────────────────────────────────────
RUN apt-get update && apt-get install -y \
    curl \
    wget \
    git \
    unzip \
    zip \
    gnupg \
    lsb-release \
    ca-certificates \
    apt-transport-https \
    software-properties-common \
    sshpass \
    openssh-client \
    jq \
    make \
    python3 \
    python3-pip \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# ── Docker CLI ─────────────────────────────────────────────────
RUN curl -fsSL https://download.docker.com/linux/debian/gpg \
      | gpg --dearmor -o /usr/share/keyrings/docker.gpg && \
    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker.gpg] \
      https://download.docker.com/linux/debian \
      $(lsb_release -cs) stable" \
      > /etc/apt/sources.list.d/docker.list && \
    apt-get update && \
    apt-get install -y docker-ce-cli && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# ── Add Jenkins user to Docker group ──────────────────────────
RUN groupadd -f docker && \
    usermod -aG docker jenkins

# ── kubectl ────────────────────────────────────────────────────
RUN curl -LO "https://dl.k8s.io/release/$(curl -sL \
      https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl" && \
    chmod +x kubectl && \
    mv kubectl /usr/local/bin/kubectl

# ── AWS CLI v2 ─────────────────────────────────────────────────
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" \
      -o "awscliv2.zip" && \
    unzip awscliv2.zip && \
    ./aws/install && \
    rm -rf awscliv2.zip aws/

# ── Terraform ──────────────────────────────────────────────────
RUN wget -O terraform.zip \
      "https://releases.hashicorp.com/terraform/1.7.0/terraform_1.7.0_linux_amd64.zip" && \
    unzip terraform.zip && \
    mv terraform /usr/local/bin/ && \
    rm terraform.zip

# ── Trivy (Security Scanner) ───────────────────────────────────
RUN wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key \
      | gpg --dearmor > /usr/share/keyrings/trivy.gpg && \
    echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] \
      https://aquasecurity.github.io/trivy-repo/deb generic main" \
      > /etc/apt/sources.list.d/trivy.list && \
    apt-get update && \
    apt-get install -y trivy && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# ── Helm ───────────────────────────────────────────────────────
RUN curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 \
      | bash

# ── Node.js 20 ─────────────────────────────────────────────────
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && \
    apt-get install -y nodejs && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# ── Maven ──────────────────────────────────────────────────────
RUN apt-get update && \
    apt-get install -y maven && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# ── SonarQube Scanner ──────────────────────────────────────────
RUN wget -O /tmp/sonar-scanner.zip \
      "https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-linux.zip" && \
    unzip /tmp/sonar-scanner.zip -d /opt/ && \
    mv /opt/sonar-scanner-5.0.1.3006-linux /opt/sonar-scanner && \
    ln -s /opt/sonar-scanner/bin/sonar-scanner /usr/local/bin/sonar-scanner && \
    rm /tmp/sonar-scanner.zip

# ── eksctl ─────────────────────────────────────────────────────
RUN curl --silent --location \
      "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" \
      | tar xz -C /tmp && \
    mv /tmp/eksctl /usr/local/bin/

# ── SSH Config ─────────────────────────────────────────────────
RUN mkdir -p /var/lib/jenkins/.ssh && \
    echo "Host *\n\
  StrictHostKeyChecking no\n\
  UserKnownHostsFile /dev/null" \
    > /var/lib/jenkins/.ssh/config && \
    chown -R jenkins:jenkins /var/lib/jenkins/.ssh && \
    chmod 700 /var/lib/jenkins/.ssh && \
    chmod 600 /var/lib/jenkins/.ssh/config

# ── Jenkins Plugins pre-install ────────────────────────────────
USER jenkins

RUN jenkins-plugin-cli --plugins \
    git \
    github \
    workflow-aggregator \
    pipeline-stage-view \
    docker-workflow \
    docker-plugin \
    ssh-agent \
    credentials-binding \
    sonar \
    kubernetes \
    aws-credentials \
    blueocean \
    build-timeout \
    timestamper \
    ws-cleanup \
    email-ext \
    slack \
    nodejs \
    maven-plugin \
    pipeline-utility-steps \
    AnsiColor

# ── Back to root for final setup ──────────────────────────────
USER root

# ── Verify all tools ───────────────────────────────────────────
RUN echo "=== Verifying Tools ===" && \
    docker  --version       && \
    kubectl version --client && \
    aws     --version        && \
    terraform --version      && \
    helm    version          && \
    node    --version        && \
    npm     --version        && \
    mvn     --version        && \
    trivy   --version        && \
    sonar-scanner --version  && \
    eksctl  version          && \
    echo "=== All tools verified! ==="

# ── Switch back to jenkins user ────────────────────────────────
USER jenkins

# ── Expose Jenkins port ────────────────────────────────────────
EXPOSE 8080 50000
