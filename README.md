# install-arch-other-drive

필자의 데탑은 2019년에 맞췄을 당시에는 저장공간 구성이 NVMe 500GB + HDD 1TB 구성이었지만, 먼지 청소도 할 겸 SSD 256GB를 더 달았다.

NVMe에 파티션을 나누었을 당시에는 하드가 /dev/sda였기 때문에 SSD는 sdb로 배정될 것이라 부트로더와 swap을 NVMe에 설치하고 SSD에 시스템 파일과 홈 폴더를 넣을 것이다. HP 놋북에는 하나의 EFI 파티션당 하나의 부트로더만 인식하였기 때문에 우분투 설치에 애로사항이 많았지만, 데탑은 그렇지 않기 때문에 nvme0n1p1에 있는 부트로더를 사용해도 된다. 애플의 퓨전 드라이브처럼 시스템 파일은 NVMe에 깔고 사용자 파일은 SSD에 까는 방법도 고안하였지만, 그러면 안정성은 둘째쳐도 병목현상이 자주 발생한다. 하지만 만약의 사태를 대비하여 NVMe에 부트로더를 두 개 만들어주는 게 좋다.

## Windows에서 파티션 설정하기

필자는 영문판 Windows를 사용하기 때문에 Disk Management(디스크 관리)에 들어가서 메인 드라이브의 파티션 공간을 8392MB 정도 줄여준다. 16584MB를 할당해도 좋지만 내장글카도 아니고 16램에서 16스왑을 할당하는 경우는 없기 때문에 8GB로도 충분하다.

## 아치 리눅스 설치하기

* 먼저 파티션을 잡아준다.

```
# mkswap /dev/nvme0n1p4
# swapon
# mkfs.ext4 /dev/sdb1
```

* 리눅스가 설치될 파티션을 마운트해준다. 여기서 nvme0n1px를 입력하면 Windows 설치부터 다시 해야 한다.

```
# mount /dev/sdb1 /mnt
```

* 부팅에 필요한 패키지를 설치한다. 부트로더는 systemd-boot를 이용하는 방법도 있지만 복잡하고 실사하는 시스템이라 자칫하다가는 벽돌이 될 수도 있어서 얌전히 grub를 설치하는 게 낫다.

```
# pacstrap /mnt base base-devel linux linux-firmware linux-firmware-qlogic linux-headers dkms nvidia-dkms nvidia pipewire-pulse efibootmgr os-prober grub nano neofetch git wget curl
```

* 부트로더를 마운트해주고 fstab에 기록한다.
```
# mkdir /mnt/efi
# mount /dev/nvme0n1p3 /mnt/efi
# genfstab -U >> /mnt
```

* chroot로 진입하여 비밀번호를 설정해주고 부트로더를 설정해준다.

```
# arch-chroot /mnt
```
