# Install Docker for Kata Containers on Ubuntu

> **Note:**
>
> - This guide assumes you have
>   [already installed the Kata Containers packages](../ubuntu-installation-guide.md).

1. Install Docker with the following commands:

   > **Notes:**
   >
   > - This step is only required if Docker is not installed on the system.
   > - Newer versions of Docker have
   >   [removed devicemapper support](https://github.com/kata-containers/documentation/issues/373)
   >   so the following commands install the latest version, which includes
   >   devicemapper support.
   > - To remove the lock on the docker package to allow it to be updated:
   >   ```sh
   >   $ sudo apt-mark unhold docker-ce
   >   ```

   ```bash
   $ sudo -E apt-get -y install apt-transport-https ca-certificates software-properties-common
   $ curl -sL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   $ arch=$(dpkg --print-architecture)
   $ sudo -E add-apt-repository "deb [arch=${arch}] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
   $ sudo -E apt-get update
   $ sudo -E apt-get -y install --allow-downgrades docker-ce='18.06.2~ce~3-0~ubuntu'
   $ sudo apt-mark hold docker-ce
   ```

   For more information on installing Docker please refer to the
   [Docker Guide](https://docs.docker.com/engine/installation/linux/ubuntu).

2. Configure Docker to use Kata Containers by default with one of the following methods:

    1. systemd

        ```bash
        $ sudo mkdir -p /etc/systemd/system/docker.service.d/
        $ cat <<EOF | sudo tee /etc/systemd/system/docker.service.d/kata-containers.conf
        [Service]
        ExecStart=
        ExecStart=/usr/bin/dockerd -D --add-runtime kata-runtime=/usr/bin/kata-runtime --default-runtime=kata-runtime
        EOF
        ```

    2. Docker `daemon.json`

        Add the following definitions to `/etc/docker/daemon.json`:

        ```json
        {
          "default-runtime": "kata-runtime",
          "runtimes": {
            "kata-runtime": {
              "path": "/usr/bin/kata-runtime"
            }
          }
        }
        ```

3. Restart the Docker systemd service with the following commands:

   ```bash
   $ sudo systemctl daemon-reload
   $ sudo systemctl restart docker
   ```

4. Run Kata Containers

   You are now ready to run Kata Containers:

   ```bash
   $ sudo docker run busybox uname -a
   ```

   The previous command shows details of the kernel version running inside the
   container, which is different to the host kernel version.
