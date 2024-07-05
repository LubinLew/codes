# Get Hardware Address(MAC)

MAC address： medium access control address，
sometimes referred to as a hardware or physical address, 
is a unique, 12-character alphanumeric attribute that is used to identify individual electronic devices on a network.

## code

```c
#include <sys/types.h>
#include <sys/ioctl.h>
#include <sys/socket.h>

#include <stdio.h>
#include <errno.h>
#include <unistd.h>
#include <string.h>

#include <net/if.h>
#include <arpa/inet.h>
#include <netinet/if_ether.h>


#define DBG(fmt, ...) printf("[%s:%d]" fmt "\n", __FUNCTION__, __LINE__, ##__VA_ARGS__)

int main(int argc, const char* argv[])
{
    const char *devname = "ens32";
    if (argc > 1) {
        devname = argv[1];
    }

    /* create a raw socket */
    int fd = socket(PF_PACKET, SOCK_RAW, htons(ETH_P_IP));
    if (fd < 0) {
        DBG("socket() failed, %s", strerror(errno));
        return -1;
    }

    /* bind socket to a particular interface */
    int ret = setsockopt(fd, SOL_SOCKET, SO_BINDTODEVICE, devname, strlen(devname));
    if (ret != 0) {
        DBG("setsockopt() failed, %s", strerror(errno));
        close(fd);
        return -1;
    }

    /* get hardware address */
    struct ifreq ifr;
    uint8_t hwaddr[ETH_ALEN];
    memset(&ifr, 0, sizeof(struct ifreq));
    strncpy(ifr.ifr_name, devname, IFNAMSIZ);
    if (ioctl(fd, SIOCGIFHWADDR, &ifr) < 0) {
        DBG("ioctl() failed, %s", strerror(errno));
        close(fd);
        return -1;
    }
    memcpy(hwaddr, ifr.ifr_hwaddr.sa_data, ETH_ALEN);

    /* print hardware address */
    printf("[%s] ", devname);
    for (int i = 0; i < ETH_ALEN; i++) {
        printf("%02X:", hwaddr[i]);
    }
    printf("\b \n");

    /* cleanup */
    close(fd);
    return 0;
}
```
