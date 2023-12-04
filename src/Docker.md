# Docker

- **Install**
    
    - Linux
        
        - 1.Script
            This method is easy. It downloads the latest ***edge*** version of docker so it's not a good choice for production environment.

            [](https://get.docker.com/)
            
            **You can run the below script from above site:**
            
            ```bash
            ~: curl -fsSL https://get.docker.com -o get-docker.sh
            ~: sh get-docker.sh
            ```
            
        - 2.Docker hub
            In this method we should follow the installation instruction from docker site step by step and install Docker on our host.
            
            **Step one : Install Docker**
            
            [Get Docker Engine - Community for Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
            
            **Step two: Post installation tasks**
            
            [Post-installation steps for Linux](https://docs.docker.com/install/linux/linux-postinstall/)
            

- **Network**
        
    - Connect/Disconnect    
        
        ```bash
        ~: docker network connect   net_name    container_name
        
        # To check which networks the container is connected
        ~: docker container inspect container_name
        
        ~: docker network disconnect net_name  container_name
        ```
        
    
    - Create
        
        ```bash
        ~: docker network create --attachable -d bridge traefik_net
        ```
        
        ```bash
        ~: docker network create --attachable --ip-range 172.25.0.0/16 --subnet 172.25.0.0/16  -d bridge traefik_net2
        ```
        
    - List
        
        **List All networks:**
        
        ```bash
        ~: docker network ls
        ```
        
        **List network details:**
        
        ```bash
        ~: docker network inspect net_name
        ```
        
        **Find what containers are connected to a specific network:**
        
        ```bash
        ~: docker network inspect -f   '{{ .Containers }}'   net_name
        ```
        
        **Get specific container ip address:**
        
        ```bash
        ~: docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'     container_name
        
        # or
        
        ~: docker container inspect -f '{{ .NetworkSettings.IPAddress}}'   container_name
        
        # or
        
        ~: docker container inspect container_name | grep  "IPAddress"
        ```
        
        List all containers with ip addresses:
        
        ```bash
        ~: docker ps -q | xargs -n 1 docker inspect --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}} {{ .Name }}' | sed 's/ \// /'
        ```
        
- DNS
    
    ***In each docker network all containers can find each other using Name Resolution mechanism NOT IP Address.***
    
    ***The name is the container name that we specify when we are creating container.***
    
- Image
    
    **To see the layers of an image:**
    ```bash
    ~: docker image history
    ```
    
    **To customiza an image and upload it to docker hub:**
    
    ```bash
    # First login to docker hub
    ~: docker login
    
    # Get your desired image
    ~: docker pull image[:tag]
    
    # Make a copy of it with new name (and tag) in local image cache 
    ~: docker image tag    current_image[:tag]    mynew_image[:tag]
    
    # Finally push it to docker hub
    ~: docker push mynew_image[:tag]
    ```