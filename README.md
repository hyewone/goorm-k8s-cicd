# AWS 환경에 k8s CI/CD 구축하기
해당 프로젝트는 KDT 쿠버네티스 전문가 양성 과정의 정규 프로젝트가 아닌 강의 수강 기간 도중 선택적으로 진행한 프로젝트입니다.
<br/>EKS가 아닌 AWS EC2에 k8s 를 설치하여 클러스터를 구성했고, Jenkins 와 Argo CD를 사용하여 CI/CD 파이프라인을 구성했습니다.

<br/>

![**CI**](resources/Untitled%20(1).png)

**CI**

![**CD**](resources/Untitled%20(2).png)

**CD**

<br/>

## TO-DO

- [x]  [EC2 생성 및 ElasticIP 설정](https://www.notion.so/AWS-k8s-CI-CD-8e13f99bcf054db2a8c8c6b337a98542?pvs=21)
- [x]  [k8s 환경 구성](https://www.notion.so/AWS-k8s-CI-CD-8e13f99bcf054db2a8c8c6b337a98542?pvs=21)
- [x]  [Jenkins 컨테이너 실행 및 빌드 환경 세팅](https://www.notion.so/AWS-k8s-CI-CD-8e13f99bcf054db2a8c8c6b337a98542?pvs=21)
- [x]  [Argo CD컨테이너 실행 및 Item 설정](https://www.notion.so/AWS-k8s-CI-CD-8e13f99bcf054db2a8c8c6b337a98542?pvs=21)
- [x]  [gitea 컨테이너 실행 및 repo 설정](https://www.notion.so/AWS-k8s-CI-CD-8e13f99bcf054db2a8c8c6b337a98542?pvs=21)
- [x]  [gitea repo 생성](https://www.notion.so/AWS-k8s-CI-CD-8e13f99bcf054db2a8c8c6b337a98542?pvs=21)
- [x]  [도커 레포지토리 생성 및 Dockerfile 작성](https://www.notion.so/AWS-k8s-CI-CD-8e13f99bcf054db2a8c8c6b337a98542?pvs=21)
- [x]  [Jenkins Pipeline 작성](https://www.notion.so/AWS-k8s-CI-CD-8e13f99bcf054db2a8c8c6b337a98542?pvs=21)
- [x]  [Jenkins Pipeline 테스트](https://www.notion.so/AWS-k8s-CI-CD-8e13f99bcf054db2a8c8c6b337a98542?pvs=21)
- [x]  [WebHook 설정](https://www.notion.so/AWS-k8s-CI-CD-8e13f99bcf054db2a8c8c6b337a98542?pvs=21)


<br/>

## DONE

### EC2 생성 및 ElasticIP 설정

- EC2 생성
    
    
    ![Untitled](resources/Untitled%20(3).png)
    
    
    ![Untitled](resources/Untitled%20(4).png)
    
- ElasticIP 생성 및 연결
    
    
    ![Untitled](resources/Untitled%20(5).png)
    
- 인스턴스 ssh 접속
    
    ```bash
    ssh -i hw_jenkins.pem centos@13.124.203.239
    ```
    

---

### k8s 환경 구성

- swap 비활성화
    
    ```bash
    # swap 비활성화
    sudo swapoff -a # 현재 시스템에 적용(리부팅하면 재설정 필요)
    sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab # 리부팅 필수
    ```
    
- iptables 커널 설정
    
    ```bash
    # iptables 커널 설정
    cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    overlay
    br_netfilter
    EOF
    
    sudo modprobe overlay
    sudo modprobe br_netfilter
    
    # 필요한 sysctl 파라미터를 설정하면, 재부팅 후에도 값이 유지된다.
    cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    net.ipv4.ip_forward                 = 1
    EOF
    
    # 재부팅하지 않고 sysctl 파라미터 적용하기
    sudo sysctl --system
    ```
    
- hostname 지정 및 hosts 파일 설정
    
    ```bash
    sudo -i
    hostnamectl set-hostname master
    cat <<EOF >> /etc/hosts
    127.0.0.1   localhost
    172.31.54.3 master
    EOF
    ```
    
- containerd 설치
    
    ```bash
    # docker 설치
    sudo rpm --import https://download.docker.com/linux/centos/gpg
    
    sudo tee /etc/yum.repos.d/docker.repo <<-'EOF'
    [dockerrepo]
    name=Docker Repository
    baseurl=https://download.docker.com/linux/centos/7/x86_64/stable/
    enabled=1
    gpgcheck=1
    gpgkey=https://download.docker.com/linux/centos/gpg
    EOF
    
    sudo systemctl enable --now containerd
    
    # Containerd configuration for Kubernetes
    cat <<EOF | sudo tee -a /etc/containerd/config.toml
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
    EOF
    
    sudo sed -i 's/^disabled_plugins \=/\#disabled_plugins \=/g' /etc/containerd/config.toml
    sudo systemctl restart containerd
    
    # 소켓이 있는지 확인한다.
    ls /var/run/containerd/containerd.sock
    ```
    
- docker 설치 (안 해도됨)
    
    ```bash
    sudo yum install -y yum-utils
    sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    
    sudo yum install docker-ce docker-ce-cli docker-buildx-plugin docker-compose-plugin
    
    sudo systemctl enable --now docker
    
    sudo docker login
    Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
    Username: hyewone
    Password: 
    WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
    Configure a credential helper to remove this warning. See
    https://docs.docker.com/engine/reference/commandline/login/#credentials-store
    
    Login Succeeded
    ```
    
- kubelet, kubeadm, kubectl 설치
    
    ```bash
    # kubelet, kubeadm, kubectl 설치
    cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
    [kubernetes]
    name=Kubernetes
    baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
    enabled=1
    gpgcheck=1
    gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
    exclude=kubelet kubeadm kubectl
    EOF
    
    # permissive 모드로 SELinux 설정(효과적으로 비활성화)
    sudo setenforce 0
    sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
    
    sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
    
    sudo systemctl enable --now kubelet
    
    kubeadm version
    kubeadm version: &version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.1", GitCommit:"4c9411232e10168d7b050c49a1b59f6df9d7ea4b", GitTreeState:"clean", BuildDate:"2023-04-14T13:20:04Z", GoVersion:"go1.20.3", Compiler:"gc", Platform:"linux/amd64"}
    kubectl version
    WARNING: This version information is deprecated and will be replaced with the output from kubectl version --short.  Use --output=yaml|json to get the full version.
    Client Version: version.Info{Major:"1", Minor:"27", GitVersion:"v1.27.1", GitCommit:"4c9411232e10168d7b050c49a1b59f6df9d7ea4b", GitTreeState:"clean", BuildDate:"2023-04-14T13:21:19Z", GoVersion:"go1.20.3", Compiler:"gc", Platform:"linux/amd64"}
    Kustomize Version: v5.0.1
    
    sudo systemctl status kubelet
    ● kubelet.service - kubelet: The Kubernetes Node Agent
       Loaded: loaded (/usr/lib/systemd/system/kubelet.service; enabled; vendor preset: disabled)
      Drop-In: /usr/lib/systemd/system/kubelet.service.d
               └─10-kubeadm.conf
       Active: activating (auto-restart) (Result: exit-code) since 수 2023-05-10 12:57:58 UTC; 4s ago
         Docs: https://kubernetes.io/docs/
      Process: 30384 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS (code=exited, status=1/FAILURE)
     Main PID: 30384 (code=exited, status=1/FAILURE)
    
     5월 10 12:57:58 ip-172-31-54-3.ap-northeast-2.compute.internal systemd[1]: ...
     5월 10 12:57:58 ip-172-31-54-3.ap-northeast-2.compute.internal systemd[1]: ...
    Hint: Some lines were ellipsized, use -l to show in full.
    ```
    
- CNI 설치
    
    ```bash
    # CNI 설치 필요
    # calico CNI install https://docs.tigera.io/calico/latest/getting-started/kubernetes/quickstart
    sudo kubeadm init --pod-network-cidr=192.168.0.0/16
    
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    
    kubectl get node
    NAME     STATUS     ROLES           AGE   VERSION
    master   NotReady   control-plane   66s   v1.27.1
    
    kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.1/manifests/tigera-operator.yaml
    kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.1/manifests/custom-resources.yaml
    
    # calico pod 가 모두 running 상태가 되어야함
    watch kubectl get pods -n calico-system
    
    NAME                                       READY   STATUS    RESTARTS   AGE
    calico-kube-controllers-789dc4c76b-wg8vf   1/1     Running   0          2m46s
    calico-node-mrps9                          1/1     Running   0          2m47s
    calico-typha-67d5697c97-ntrwg              1/1     Running   0          2m47s
    csi-node-driver-g5pj5                      2/2     Running   0          2m46s
    
    k get node
    NAME     STATUS   ROLES           AGE   VERSION
    master   Ready    control-plane   11m   v1.27.1
    ```
    
- kubectl 자동완성 및 alias 설정
    
    ```bash
    sudo yum install bash-completion
    source /usr/share/bash-completion/bash_completion
    echo 'source <(kubectl completion bash)' >>~/.bashrc
    echo 'alias k=kubectl' >>~/.bashrc
    echo 'complete -o default -F __start_kubectl k' >>~/.bashrc
    ```
    
- Taint 제거
    
    ```bash
    # master node 에 스케줄링 하기 위해
    kubectl taint nodes --all node-role.kubernetes.io/control-plane-
    kubectl taint nodes --all node-role.kubernetes.io/master-
    ```
    

---

### Jenkins 컨테이너 실행 및 빌드 환경 세팅

- Jenkins 컨테이너 실행
    - 호스트 도커 sock 을 Jenkins 컨테이너에서 사용하도록 설정 필요
    - jenkins-blueocean (jenkins 서버) 컨테이너 실행 시 로컬 볼륨을 매핑하는 -v 옵션을 주었기에 컨테이너가 중지 후 restart 되어도 jenkins 관련 이전 설정 남아 있음
    
    ```bash
    # https://www.jenkins.io/doc/book/installing/docker/#downloading-and-running-jenkins-in-docker
    
    docker network create jenkins
    
    docker run \
      --name jenkins-docker \
      --rm \
      --detach \
      --privileged \
      --network jenkins \
      --network-alias docker \
      --env DOCKER_TLS_CERTDIR=/certs \
      --volume jenkins-docker-certs:/certs/client \
      --volume jenkins-data:/var/jenkins_home \
      --publish 2376:2376 \
      docker:dind \
      --storage-driver overlay2
    
    ---dockerfile--------------------------------------------------------------------
    
    FROM jenkins/jenkins:2.387.3
    USER root
    RUN apt-get update && apt-get install -y lsb-release
    RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
      https://download.docker.com/linux/debian/gpg
    RUN echo "deb [arch=$(dpkg --print-architecture) \
      signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
      https://download.docker.com/linux/debian \
      $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
    RUN apt-get update && apt-get install -y docker-ce-cli
    USER jenkins
    RUN jenkins-plugin-cli --plugins "blueocean docker-workflow"
    
    ---------------------------------------------------------------------------------
    
    docker build -t myjenkins-blueocean:2.387.3-1 .
    
    docker run \
      --name jenkins-blueocean \
      --restart=on-failure \
      --detach \
      --network jenkins \
      --env DOCKER_HOST=tcp://docker:2376 \
      --env DOCKER_CERT_PATH=/certs/client \
      --env DOCKER_TLS_VERIFY=1 \
      --publish 8080:8080 \
      --publish 50000:50000 \
      --volume jenkins-data:/var/jenkins_home \
      --volume jenkins-docker-certs:/certs/client:ro \
      myjenkins-blueocean:2.387.3-1
    
    docker exec -it contianerId bash
    
    # Jenkins Container 에서 도커 테스트
    docker run hello-world
    Unable to find image 'hello-world:latest' locally
    latest: Pulling from library/hello-world
    719385e32844: Pull complete 
    Digest: sha256:fc6cf906cbfa013e80938cdf0bb199fbdbb86d6e3e013783e5a766f50f5dbce0
    Status: Downloaded newer image for hello-world:latest
    
    Hello from Docker!
    This message shows that your installation appears to be working correctly.
    ```
    
- AWS EC2 보안그룹 인바운드 규칙 추가
    
    
    ![Untitled](resources/Untitled%20(6).png)
    
- Jenkins 컨테이너 접속 확인
    
    
    ![Untitled](resources/Untitled%20(7).png)
    
- Jenkins 환경 설정
    - 빌드에 필요한 플러그인 및 패키지 설치
    - git, ant, docker 확인
    
    ```bash
    apt install ant
    ant -v
    Apache Ant(TM) version 1.10.9 compiled on December 25 1969
    ```
    

---

### Argo CD컨테이너 실행 및 Item 설정

- argocd namespace 생성
    
    ```bash
    kubectl create namespace argocd
    ```
    
- Argo CD 컨테이너 실행
    
    ```bash
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
    
    kubectl get svc,pod -n argocd
    NAME                                              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
    service/argocd-applicationset-controller          ClusterIP   10.108.203.119   <none>        7000/TCP,8080/TCP            94s
    service/argocd-dex-server                         ClusterIP   10.100.91.72     <none>        5556/TCP,5557/TCP,5558/TCP   94s
    service/argocd-metrics                            ClusterIP   10.103.98.125    <none>        8082/TCP                     94s
    service/argocd-notifications-controller-metrics   ClusterIP   10.107.124.8     <none>        9001/TCP                     93s
    service/argocd-redis                              ClusterIP   10.104.177.50    <none>        6379/TCP                     93s
    service/argocd-repo-server                        ClusterIP   10.100.10.177    <none>        8081/TCP,8084/TCP            93s
    service/argocd-server                             ClusterIP   10.101.95.43     <none>        80/TCP,443/TCP               93s
    service/argocd-server-metrics                     ClusterIP   10.99.18.8       <none>        8083/TCP                     93s
    
    NAME                                                    READY   STATUS    RESTARTS   AGE
    pod/argocd-application-controller-0                     1/1     Running   0          93s
    pod/argocd-applicationset-controller-6bf657d56d-694zg   1/1     Running   0          93s
    pod/argocd-dex-server-5778f5b7b4-tbjqb                  1/1     Running   0          93s
    pod/argocd-notifications-controller-5d49c977f5-jpj92    1/1     Running   0          93s
    pod/argocd-redis-77bf5b886-cv8wm                        1/1     Running   0          93s
    pod/argocd-repo-server-7bd9b7df75-pbksk                 1/1     Running   0          93s
    pod/argocd-server-f84cf8b46-j2fqb                       1/1     Running   0          93s
    ```
    
- 서비스 NodePort 로 변경
    
    ```bash
    # 외부에서 접속할 수 있도록 NodePort 타입으로 변경
    kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
    kubectl get svc argocd-server -n argocd -w
    NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
    argocd-server   ClusterIP   10.101.95.43   <none>        80/TCP,443/TCP   2m32s
    argocd-server   NodePort    10.101.95.43   <none>        80:32383/TCP,443:30827/TCP   5m16s
    ```
    
- AWS EC2 보안그룹 인바운드 규칙 추가
    
   
    ![Untitled](resources/Untitled%20(8).png)
    
- Argo CD 접속 테스트
    
    ```bash
    # default 계정 admin
    # 패스워드 명령어
    kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
    ```
    
    
    ![Untitled](resources/Untitled%20(9).png)
    
- Argo CD Item 추가
    
    
    ![Untitled](resources/Untitled%20(10).png)
    
   
    ![Untitled](resources/Untitled%20(11).png)
    
    
    ![Untitled](resources/Untitled%20(12).png)
    

---

### gitea 컨테이너 실행 및 repo 설정

- gitea  컨테이너 실행
    
    ```bash
    sudo docker run -d --name=gitea -p 3000:3000 -v /var/lib/gitea:/data gitea/gitea:latest
    
    sudo docker ps
    CONTAINER ID   IMAGE                COMMAND                   CREATED          STATUS         PORTS                                                                                      NAMES
    105deae4e7a8   gitea/gitea:latest   "/usr/bin/entrypoint…"   10 seconds ago   Up 9 seconds   22/tcp, 0.0.0.0:3000->3000/tcp, :::3000->3000/tcp                                          gitea
    53f1701b2312   jenkins/jenkins      "/usr/bin/tini -- /u…"   2 hours ago      Up 2 hours     0.0.0.0:8080->8080/tcp, :::8080->8080/tcp, 0.0.0.0:50000->50000/tcp, :::50000->50000/tcp   jenkins-server
    ```
    
- AWS EC2 보안그룹 인바운드 규칙 추가
    
    
    ![Untitled](resources/Untitled%20(13).png)
    
- gitea 접속 테스트
    
    
    ![Untitled](resources/Untitled%20(14).png)
    

---

### gitea repo 생성

- 초기 설정
    
    
    ![Untitled](resources/Untitled%20(15).png)
    
- tomcat-project repo 생성
    
    
    ![Untitled](resources/Untitled%20(16).png)
    
    ```bash
    mkdir tomcat-project
    touch README.md
    git init
    git checkout -b main
    git add README.md
    git config --global user.email "won5854@naver.com"
    git config --global user.name "hyewon"
    git commit -m "first commit"
    git remote add origin http://13.124.203.239:3000/montoring/tomcat-project.git
    git push -u origin main
    ```
    
    ```bash
    scp -i hw_jenkins.pem apache-tomcat-8.5.88-src.tar.gz centos@13.124.203.239:/home/centos/tomcat-project
    ```
    
    ```bash
    ls
    README.md  apache-tomcat-8.5.88-src.tar.gz
    
    gunzip apache-tomcat-8.5.88-src.tar.gz 
    ls
    README.md  apache-tomcat-8.5.88-src.tar
    
    tar xf apache-tomcat-8.5.88-src.tar 
    ls
    README.md  apache-tomcat-8.5.88-src  apache-tomcat-8.5.88-src.tar
    
    mv apache-tomcat-8.5.88-src/* .
    ls
    BUILDING.txt     README.md                     build.properties.default  res
    CONTRIBUTING.md  RELEASE-NOTES                 build.properties.release  test
    KEYS             RUNNING.txt                   build.xml                 webapps
    LICENSE          apache-tomcat-8.5.88-src      conf
    MERGE.txt        apache-tomcat-8.5.88-src.tar  java
    NOTICE           bin                           modules
    
    rm -rf apache-tomcat-8.5.88-src
    rm -rf apache-tomcat-8.5.88-src.tar 
    ls
    BUILDING.txt     MERGE.txt      RUNNING.txt               build.xml  res
    CONTRIBUTING.md  NOTICE         bin                       conf       test
    KEYS             README.md      build.properties.default  java       webapps
    LICENSE          RELEASE-NOTES  build.properties.release  modules
    
    git add --all
    git commit -m "tomcat unzip commit"
    ```
    
    
    ![Untitled](resources/Untitled%20(17).png)
    
- argocd-yaml repo 생성
    - deployment.yaml, service-nodeport.yaml 파일 생성
    
    
    ![Untitled](resources/Untitled%20(18).png)
    
    ```bash
    sudo yum install git
    
    touch README.md
    git init
    git checkout -b main
    git add README.md
    git config --global user.email "won5854@naver.com"
    git config --global user.name "hyewon"
    git commit -m "first commit"
    git remote add origin http://13.124.203.239:3000/montoring/argocd-yaml.git
    git push -u origin main
    ```
    
    ```bash
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: tomcat-deployment
      labels:
        argocd: deploy
    spec:
      replicas: 2
      selector:
        matchLabels:
          argocd: deploy
      template:
        metadata:
          labels:
            argocd: deploy
        spec:
          containers:
          - name: tomcat
            image: hyewone/mentoring_project:latest
            ports:
            - containerPort: 80
    
    ---
    
    apiVersion: v1
    kind: Service
    metadata:
      name: service-nodeport
    spec:
      type: NodePort
      selector:
        argocd: deploy
      ports:
        - port: 80
          targetPort: 8080
          nodePort: 31111
    ```
    
   
    ![Untitled](resources/Untitled%20(19).png)
    
- 노드 식별 라벨 생성
    
    ```bash
    k label node master argoCD=deploy
    
    k get node -L argoCD
    NAME     STATUS   ROLES           AGE     VERSION   ARGOCD
    master   Ready    control-plane   8m42s   v1.27.1   deploy
    ```
    

---

### 도커 레포지토리 생성 및 Dockerfile 작성

- 도커 레포지토리 생성
    
    
    ![Untitled](resources/Untitled%20(20).png)
    
- Dockerfile 작성
    
    
    ![Untitled](resources/Untitled%20(21).png)
    

---

### Jenkins Pipeline 작성

```groovy
pipeline {
    agent any
    
    environment {
        GITEA_DOMAIN = "13.124.203.239:3000/montoring/"
        APP_REPO = "tomcat-project.git"
        YAML_REPO = "argocd-yaml.git"
        DOCKER_IMAGE = "hyewone/mentoring_project"
        DOCKER_IMAGE_TAG = "v${env.BUILD_ID}"
    }
    
    stages {
       
        stage('Build') {
            steps {
                // Get some code from a GitHub repository
                dir('app-repo') {
                    git branch: 'main', url: 'http://' + GITEA_DOMAIN + APP_REPO 
               
                    sh 'ant clean'
                    sh 'ant package'
                    sh 'ant deploy'
                }
            }
        }
        
        stage('Upload to Docker Registry') {
            steps {
                // 도커 이미지 빌드 및 업로드
                dir('app-repo') {
                    
                    withCredentials([usernamePassword(credentialsId: 'docker_login_info', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_IMAGE_TAG} ."
                        sh 'docker login -u ${USERNAME} -p ${PASSWORD}'
                        sh "docker push ${DOCKER_IMAGE}:${DOCKER_IMAGE_TAG}"
                    }
                }
            }
        }
        
        stage('Update ArgoCD YAML') {
            steps {
                // ArgoCD YAML 파일 체크아웃
                dir('yarml-repo') {
                    
                    git branch: 'main', url: 'http://' + GITEA_DOMAIN + YAML_REPO
                                
                    script {
                        // YAML 파일에서 이미지 버전 변경
                        sh 'sed -i "s|' + DOCKER_IMAGE +':.*|' + DOCKER_IMAGE + ':' + DOCKER_IMAGE_TAG +'|" deployment.yaml'
                    }
                   
                    // 변경된 YAML 파일 커밋 및 푸시
                    withCredentials([usernamePassword(credentialsId: 'git_email', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh 'git config user.email ${USERNAME}'
                    }
                    withCredentials([usernamePassword(credentialsId: 'git_name', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh 'git config user.name ${USERNAME}'
                    }
                    sh 'git add deployment.yaml'
                    sh 'git commit -m "Update image version"'
                    
                    withCredentials([usernamePassword(credentialsId: 'gitea_token', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh 'git push http://${USERNAME}:${PASSWORD}@${GITEA_DOMAIN}${YAML_REPO}'
                    }
                }
            }
        }
    }

    post {
        failure {
            slackSend channel: '#k8s-cicd',
            color: 'danger',
            message: "${DOCKER_IMAGE}:${DOCKER_IMAGE_TAG} Build failed!",
            tokenCredentialId: 'slack_webhook'
        }

        success {
            slackSend channel: '#k8s-cicd',
            color: 'good',
            message: "${DOCKER_IMAGE}:${DOCKER_IMAGE_TAG} Build successful!",
            tokenCredentialId: 'slack_webhook'
        }
    }
}
```

- agent any : build 하는 agent
- environment :
- stage('Build') : ant build 단계
- stage('Upload to Docker Registry') : 도커 이미지 생성 및 도커 이미지 업로드 단계
- stage('Update ArgoCD YAML') : 도커 레포지토리와 이미지 버전 비교하여, yaml 파일의 버전을 최신 버전으로 update commit

---

### Jenkins Pipeline 테스트


![Untitled](resources/Untitled%20(22).png)
![Untitled](resources/Untitled%20(23).png)
![Untitled](resources/Untitled%20(24).png)
![Untitled](resources/Untitled%20(25).png)

```bash
k get pod,svc -n argocd-deploy 
NAME                                     READY   STATUS    RESTARTS   AGE
pod/tomcat-deployment-8679c89c8f-2nkcp   1/1     Running   0          16m
pod/tomcat-deployment-8679c89c8f-55vf7   1/1     Running   0          16m

NAME                       TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
service/service-nodeport   NodePort   10.98.43.62   <none>        80:31111/TCP   19h
```


    ![Untitled](resources/Untitled%20(26).png)

---

### Webhook 설정

- Jenkins Generic Webhook Trigger Plugin 설치
    
   
    ![Untitled](resources/Untitled%20(27).png)
    
- Jenkins API Token 발급
    
    
    ![Untitled](resources/Untitled%20(28).png)
    
- gitea Webhook  설정
    
    
    ![Untitled](resources/Untitled%20(29).png)
    
- Jenkins 프로젝트 Generic Webhook Trigger 설정
    
    ![Untitled](resources/Untitled%20(30).png)
    
- Webhook 전달 시험
    
    ![Untitled](resources/Untitled%20(31).png)
    
- build
    
    ![Untitled](resources/Untitled%20(32).png)
    
