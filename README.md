# fedora-nvidia-blackwell
Installation du driver Nvidia sous Fedora avec les spécificité pour Blackwell et un GPU intégré.

# Nvidia et Fedora
Mis à jour le 19 avril 2025

## Demander l’installation du pilote de noyau libre
Étape indispensable avec les GPU Blackwell (série 5000)
Il ne s’agit pas du pilote libre "nouveau" mais bien du kernel nvidia open source (la partie usespace du pilote reste proprio)
`sudo sh -c 'echo "%_with_kmod_nvidia_open 1" > /etc/rpm/macros.nvidia-kmod'`

## Désactiver le pilote libre "nouveau"
Éditer /etc/default/grub et ajouter
`module_blacklist=nouveau rd.driver.blacklist=nouveau modprobe.blacklist=nouveau`
à GRUB_CMDLINE_LINUX
Le premier empêche le noyau de charger le pilote, le second empêche le ramdisk de le faire et le dernier empêche le système de le faire.
Tant que vous êtes là ajouter à la même ligne :
`nvidia_drm.modeset=1 nvidia_drm.fbdev=1`
Ça ne devrait plus être utile avec Wayland, mais certaines versions du pilote ont eu des régressions qui nécessite ces paramètre.
`nvidia.NVreg_RegistryDwords=EnableBrightnessControl=1`
Ça c’est pour contrôler la luminosité de l’écran par DCC
Enfin on enregistre ça.
`sudo grub2-mkconfig -o /boot/grub2/grub.cfg`
Et pour éviter un chargement involontaire du mauvais pilote par la suite :
`sudo sh -c 'echo "install nouveau /usr/bin/false" > /etc/modprobe.d/nvidia-blacklist.conf'`
Enfin, supprimer "nouveau" pour X11
`sudo dnf remove xorg-x11-drv-nouveau`

## Installation
`sudo dnf install akmod-nvidia xorg-x11-drv-nvidia-cuda vulkan xorg-x11-drv-nvidia-cuda-libs nvidia-utils`

## Compilation
Construction du kernel (ça le fera au redémarrage, mais au moins on peut vérifier que tout est OK)
`sudo akmods --kernels $(uname -r) --rebuild`
`modinfo -l nvidia`
Si vous êtes bon, ça affiche « Dual MIT/GPL »

## Charger les bons pilotes dès le boot
`sudo sh -c 'echo "force_drivers+=\" nvidia nvidia_modeset nvidia_uvm nvidia_drm \"" > /etc/dracut.conf.d/nvidia.conf'`
Reconstruire le ramdisk
`sudo dracut -fHv`

## Reboot
`reboot`

## Vérifier les ROP
Si vous avez un GPU Blackwell de début 2025, il vaudrait mieux vérifier qu’il est entier…
`sudo dnf copr enable ilyaz/LACT`
`sudo dnf install lact`
`lact`

## Utiliser le bon GPU avec Wayland
S’applique si vous avez un GPU intégré au CPU.
Le guide suivant est très bien.
https://9to5linux.com/how-to-switch-primary-gpu-to-nvidia-on-wayland-for-kde-plasma-and-gnome
Simplement pour KDE, il est possible de le faire system wide dans "/etc/environment" plutôt que par utilisateur.
Il semble aussi que certaines version de KWIN utilisent aussi
`KWIN_DRM_DEVICES="/dev/dri/card0:/dev/dri/card1"` (avec card0 et card1 correspondants aux valeurs trouvées dans le tuto "9to5linux").


## Sources
https://rpmfusion.org/Howto/NVIDIA#Kernel_Open
https://doc.fedora-fr.org/wiki/Carte_graphique_NVIDIA_:_installation_des_pilotes_propriétaires
https://discussion.fedoraproject.org/t/nvidia-how-to-switch-to-the-open-source-kernel-module/126362/4
https://bbs.archlinux.org/viewtopic.php?pid=2143543#p2143543
https://github.com/ilya-zlobintsev/LACT
https://9to5linux.com/how-to-switch-primary-gpu-to-nvidia-on-wayland-for-kde-plasma-and-gnome
https://discuss.kde.org/t/plasma-breaks-on-wayland-with-nvidia-proprietary-drivers-egpu/8749
