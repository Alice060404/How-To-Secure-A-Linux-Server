# 如何加固 Linux 服务器

一份持续完善的 Linux 服务器加固操作指南；希望它还能让你了解一些安全知识，以及安全为何重要。

[![CC-BY-SA](https://i.creativecommons.org/l/by-sa/4.0/88x31.png)](#license)

<a id="table-of-contents"></a>
## 目录

- [简介](#introduction)
  - [指南目标](#guide-objective)
  - [为什么要加固服务器](#why-secure-your-server)
  - [为什么还要再写一份指南](#why-yet-another-guide)
  - [其他指南](#other-guides)
  - [待办/待添加内容](#to-do--to-add)
- [指南概述](#guide-overview)
  - [关于本指南](#about-this-guide)
  - [我的使用场景](#my-use-case)
  - [编辑配置文件——给想省事的人](#editing-configuration-files---for-the-lazy)
  - [参与贡献](#contributing)
- [开始之前](#before-you-start)
  - [明确你的原则](#identify-your-principles)
  - [选择 Linux 发行版](#picking-a-linux-distribution)
  - [安装 Linux](#installing-linux)
  - [安装前/后的要求](#prepost-installation-requirements)
  - [其他重要注意事项](#other-important-notes)
  - [使用 Ansible playbook 加固你的 Linux 服务器](#using-ansible-playbooks-to-secure-your-linux-server)
- [SSH 服务器](#the-ssh-server)
  - [修改 SSH 前的重要说明](#important-note-before-you-make-ssh-changes)
  - [SSH 公钥/私钥](#ssh-publicprivate-keys)
  - [为 AllowGroups 创建 SSH 组](#create-ssh-group-for-allowgroups)
  - [加固 `/etc/ssh/sshd_config`](#secure-etcsshsshd_config)
  - [移除短 Diffie-Hellman 密钥](#remove-short-diffie-hellman-keys)
  - [为 SSH 配置 2FA/MFA](#2famfa-for-ssh)
- [基础设置](#the-basics)
  - [限制 sudo 的使用者](#limit-who-can-use-sudo)
  - [限制 su 的使用者](#limit-who-can-use-su)
  - [使用 FireJail 在沙箱中运行应用程序](#run-applications-in-a-sandbox-with-firejail)
  - [NTP 客户端](#ntp-client)
  - [保护 /proc](#securing-proc)
  - [强制账户使用安全密码](#force-accounts-to-use-secure-passwords)
  - [自动安全更新与提醒](#automatic-security-updates-and-alerts)
  - [更安全的随机熵池（WIP）](#more-secure-random-entropy-pool-wip)
  - [添加胁迫/备用/伪密码登录安全系统](#add-panicsecondaryfake-password-login-security-system)
- [网络](#the-network)
  - [使用 UFW（Uncomplicated Firewall）配置防火墙](#firewall-with-ufw-uncomplicated-firewall)
  - [使用 PSAD 和 iptables 进行入侵检测与防御](#iptables-intrusion-detection-and-prevention-with-psad)
  - [使用 Fail2Ban 进行应用程序入侵检测与防御](#application-intrusion-detection-and-prevention-with-fail2ban) 
  - [使用 CrowdSec 进行应用程序入侵检测与防御](#application-intrusion-detection-and-prevention-with-crowdsec)
- [审计](#the-auditing)
  - [使用 AIDE 监控文件/文件夹完整性（WIP）](#filefolder-integrity-monitoring-with-aide-wip)
  - [使用 ClamAV 进行防病毒扫描（WIP）](#anti-virus-scanning-with-clamav-wip)
  - [使用 Rkhunter 检测 Rootkit（WIP）](#rootkit-detection-with-rkhunter-wip)
  - [使用 chrootkit 检测 Rootkit（WIP）](#rootkit-detection-with-chrootkit-wip)
  - [logwatch——系统日志分析与报告工具](#logwatch---system-log-analyzer-and-reporter)
  - [ss——查看服务器正在监听的端口](#ss---seeing-ports-your-server-is-listening-on)
  - [Lynis——Linux 安全审计](#lynis---linux-security-auditing)
  - [OSSEC——主机入侵检测](#ossec---host-intrusion-detection)
- [高风险操作区](#the-danger-zone)
- [其他内容](#the-miscellaneous)
  - [结合 Google 使用 MSMTP（简易 Sendmail）](#the-simple-way-with-msmtp)
  - [使用隐式 TLS 将 Gmail 和 Exim4 配置为 MTA](#gmail-and-exim4-as-mta-with-implicit-tls)
  - [单独的 iptables 日志文件](#separate-iptables-log-file)
- [其他信息](#left-over)
  - [联系我](#contacting-me)
  - [实用链接](#helpful-links)
  - [致谢](#acknowledgments)
  - [许可证与版权](#license-and-copyright)

（目录由 [nGitHubTOC](https://imthenachoman.github.io/nGitHubTOC/) 生成）

<a id="introduction"></a>
## 简介

<a id="guide-objective"></a>
### 指南目标

本指南旨在教你如何加固 Linux 服务器。

你可以采取许多措施来加固 Linux 服务器，本指南将尽可能多地介绍这些措施。随着我的学习，以及大家的[贡献](#contributing)，还会加入更多主题和资料。

本指南的 Ansible playbook 可参见[使用 Ansible 加固 Linux 服务器](https://github.com/moltenbit/How-To-Secure-A-Linux-Server-With-Ansible)，由 [moltenbit](https://github.com/moltenbit) 提供。

([目录](#table-of-contents))

<a id="why-secure-your-server"></a>
### 为什么要加固服务器

我假定你使用本指南，是因为你已经理解良好的安全防护为何重要。这个话题本身十分庞大，对其展开讨论超出了本指南的范围。如果你还不知道答案，我建议先研究这个问题。

从宏观上说，服务器等设备一旦进入公共领域——也就是对外界可见——就会成为恶意行为者的目标。未受保护的设备会成为他们的游乐场：他们可能想获取你的数据，也可能想把你的服务器用作其大规模 DDOS 攻击的另一个节点。

更糟的是，如果没有良好的安全防护，你可能永远都不知道服务器是否已被攻破。恶意行为者可能未经授权访问了你的服务器并复制数据，却没有更改任何内容，因此你无从察觉。你的服务器也可能参与过 DDOS 攻击，而你同样毫不知情。看看新闻中的许多大规模数据泄露事件：相关公司往往直到恶意行为者离开很久之后，才发现数据泄露或入侵。

与普遍看法相反，恶意行为者并不总是想更改某些内容，或[锁住你的数据以索取赎金](https://en.wikipedia.org/wiki/Ransomware)。有时，他们只想把你服务器上的数据收入其数据仓库（大数据蕴含巨大利益），或暗中利用你的服务器实现恶意目的。

([目录](#table-of-contents))

<a id="why-yet-another-guide"></a>
### 为什么还要再写一份指南

本指南可能看起来重复且没有必要，因为网上已有无数文章介绍[如何加固 Linux](https://duckduckgo.com/?q=how+to+secure+linux&t=ffab&atb=v151-7&ia=web)。但这些信息分散在不同文章中，涵盖的内容和讲解方式各不相同。谁有时间翻遍数百篇文章呢？

在研究如何搭建自己的 Debian 系统时，我一直在记笔记。最后我意识到，把原本知道的内容和新学到的知识结合起来，已经足以构成一份操作指南。我决定把它放到网上，希望能帮助其他人**学习**并**节省时间**。

我从未找到一份涵盖所有内容的指南——本指南就是我的尝试。

本指南介绍的许多内容可能相当基础或琐碎，但我们大多数人并非每天都安装 Linux，很容易忘记这些基本事项。

([目录](#table-of-contents))

<a id="other-guides"></a>
### 其他指南

专家、行业领导者和各 Linux 发行版自身都提供了许多指南。把这些指南的全部内容纳入此处既不现实，有时还会侵犯版权。我建议你先查看它们，再开始阅读本指南。

- [互联网安全中心（CIS）](https://www.cisecurity.org/)提供了详尽、受业界信赖的[基准](https://www.cisecurity.org/cis-benchmarks/)，其中包含加固多种 Linux 发行版的分步说明。详情请查看其[关于我们](https://www.cisecurity.org/about-us/)页面。我的建议是先通读当前这份指南，**然后**再阅读 CIS 指南；这样，只要两者存在差异，就以 CIS 的建议为准。
- 有关特定发行版的加固/安全指南，请查阅该发行版的文档。
- https://security.utexas.edu/os-hardening-checklist/linux-7 - Red Hat Enterprise Linux 7 加固核对清单
- https://cloudpro.zone/index.php/2018/01/18/debian-9-3-server-setup-guide-part-1/ - # Debian 9.3 服务器设置指南
- https://blog.vigilcode.com/2011/04/ubuntu-server-initial-security-quick-secure-setup-part-i/ - Ubuntu Server 初始安全指南
- https://www.tldp.org/LDP/sag/html/index.html
- https://seifried.org/lasg/
- https://news.ycombinator.com/item?id=19178964
- https://wiki.archlinux.org/index.php/Security - 许多人也推荐了这份指南
- https://securecompliance.co/linux-server-hardening-checklist/

([目录](#table-of-contents))

<a id="to-do--to-add"></a>
### 待办/待添加内容

- [ ] [Fail2ban 自定义 jail](#custom-jails)
- [ ] MAC（强制访问控制）和 Linux 安全模块（LSM）
   - https://wiki.archlinux.org/index.php/security#Mandatory_access_control
   - Security-Enhanced Linux / SELinux
       - https://en.wikipedia.org/wiki/Security-Enhanced_Linux
       - https://linuxtechlab.com/beginners-guide-to-selinux/
       - https://linuxtechlab.com/replicate-selinux-policies-among-linux-machines/
       - https://teamignition.us/how-to-stop-being-a-scrub-and-learn-to-use-selinux.html
   - AppArmor
       - https://wiki.archlinux.org/index.php/AppArmor
       - https://security.stackexchange.com/questions/29378/comparison-between-apparmor-and-selinux
        - http://www.insanitybit.com/2012/06/01/why-i-like-apparmor-more-than-selinux-5/
- [ ] 磁盘加密
- [ ] Rkhunter 和 chrootkit
    - http://www.chkrootkit.org/
    - http://rkhunter.sourceforge.net/
    - https://www.cyberciti.biz/faq/howto-check-linux-rootkist-with-detectors-software/
    - https://www.tecmint.com/install-rootkit-hunter-scan-for-rootkits-backdoors-in-linux/
- [ ] 传送/备份日志 - https://news.ycombinator.com/item?id=19178681
- [ ] CIS-CAT - https://learn.cisecurity.org/cis-cat-landing-page
- [ ] debsums - https://blog.sleeplessbeastie.eu/2015/03/02/how-to-verify-installed-packages/

([目录](#table-of-contents))

<a id="guide-overview"></a>
## 指南概述

<a id="about-this-guide"></a>
### 关于本指南

本指南……

- ……**仍在**不断完善。
- ……**是一份**侧重于**家用** Linux 服务器的指南。这里的所有概念和建议也适用于规模更大或更专业的环境，但这些使用场景需要更高级、更专门的配置，超出了本指南的范围。
- ……**不会**教你 Linux 基础、如何[安装 Linux](#installing-linux) 或如何使用 Linux。如果你刚接触 Linux，请查看 https://linuxjourney.com/ 。
- ……**旨在**[不受特定 Linux 发行版限制](#picking-a-linux-distribution)。
- ……**不会**教你安全方面需要了解的一切，也不会涉及系统/服务器安全的所有层面。例如，物理安全不在本指南的范围内。
- ……**不会**讲解程序/工具的工作方式，也不会深入研究其所有细枝末节。本指南引用的大多数程序/工具都非常强大，并且高度可配置。我们的目标只是介绍最基本的必要内容——足以激起你的兴趣，让你愿意进一步学习。
- ……**旨在**通过提供可复制粘贴的代码来简化操作。粘贴前可能需要修改命令，因此请准备好你喜欢的[文本编辑器](https://notepad-plus-plus.org/)。
- ……**是**按照我认为符合逻辑的顺序组织的——例如先加固 SSH，再安装防火墙。因此，本指南原本是供你按呈现顺序执行的，但并非必须如此。如果采用不同顺序，请务必谨慎——有些章节要求先完成前面的章节。

([目录](#table-of-contents))

<a id="my-use-case"></a>
### 我的使用场景

服务器有许多类型，使用场景也各不相同。虽然我希望本指南尽可能通用，但其中有些内容可能并不适用于所有或其他使用场景。阅读本指南时，请自行作出合理判断。

为了说明本指南中许多主题所处的实际环境，我的使用场景/配置如下：

- 一台台式机级别的计算机……
- 配有一块 NIC……
- 连接到消费级路由器……
- 使用 ISP 提供的动态 WAN IP……
- WAN 和 LAN 均使用 IPV4……
- LAN 使用 [NAT](https://en.wikipedia.org/wiki/Network_address_translation)……
- 我希望能从未知计算机和未知地点（例如朋友家）通过 SSH 远程连接到它。

([目录](#table-of-contents))

<a id="editing-configuration-files---for-the-lazy"></a>
### 编辑配置文件——给想省事的人

我很懒，只要没有必要，就不喜欢手动编辑文件。我假定其他人也和我一样。:)

因此，只要条件允许，我就会提供 `code` 片段，用于快速完成所需操作，例如在配置文件中添加或更改一行。

这些 `code` 片段使用 `echo`、`cat`、`sed`、`awk` 和 `grep` 等基本命令。`code` 片段的工作原理，例如每条命令或每个部分的作用，超出了本指南的范围——善用 `man` 页面。

**注意**：这些 `code` 片段不会验证或确认更改是否成功应用——例如相应的行是否真的已添加或更改。验证工作就交给你了。本指南的操作步骤确实包括备份所有将被更改的文件。

并非所有更改都能通过 `code` 片段自动完成；有些更改仍需使用传统方式手动编辑。例如，你不能简单地向 [INI](https://en.wikipedia.org/wiki/INI_file) 类型的文件末尾追加一行。请使用你[喜欢的](https://en.wikipedia.org/wiki/Vi) Linux 文本编辑器。

([目录](#table-of-contents))

<a id="contributing"></a>
### 参与贡献

我把本指南放在 [GitHub](http://www.github.com) 上，是为了便于协作。参与贡献的人越多，本指南就会变得越好、越完整。

要参与贡献，你可以 fork 仓库并提交 pull request，也可以提交一个[新 issue](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/issues/new)。

([目录](#table-of-contents))

<a id="before-you-start"></a>
## 开始之前

<a id="identify-your-principles"></a>
### 明确你的原则

开始之前，你需要明确自己的原则。你的[威胁模型](https://en.wikipedia.org/wiki/Threat_model)是什么？可以考虑以下问题：

- 你为什么要加固服务器？
- 你希望具备多高的安全性，又有哪些安全措施是你不想采用的？
- 为了安全，你愿意牺牲多少便利性？反过来又愿意为了便利牺牲多少安全性？
- 你想防范哪些威胁？你的具体情况有哪些特点？例如：
  - 他人能否通过物理接触你的服务器/网络发起攻击？
  - 你是否会在路由器上开放端口，以便从家外访问服务器？
  - 你是否会在服务器上托管文件共享，并将其挂载到台式机级别的计算机上？这台计算机感染恶意软件、进而感染服务器的可能性有多大？
 - 如果安全措施将你挡在自己的服务器之外，你是否有恢复手段？例如，你[禁用了 root 登录](#disable-root-login)，或[为 GRUB 设置了密码保护](#password-protect-grub)。

这些只是需要考虑的**少数几个问题**。开始加固服务器之前，你需要理解自己要防范什么以及为什么要防范，从而知道需要采取哪些措施。

([目录](#table-of-contents))

<a id="picking-a-linux-distribution"></a>
### 选择 Linux 发行版

本指南力求不受特定发行版限制，让用户可以使用自己想要的[任何发行版](https://distrowatch.com/)。不过，仍需注意以下几点：

你需要选择一种……

- ……**稳定的**发行版。除非你喜欢在凌晨 2 点调试问题，否则你一定不希望一次[无人值守升级](#automatic-security-updates-and-alerts)，或手动的软件包/系统更新，导致服务器无法运行。不过，这也意味着你能接受不使用最新、最强、最前沿的软件。
- ……**及时提供安全补丁的**发行版。你可以加固服务器上的一切，但如果正在运行的核心操作系统或应用程序存在已知漏洞，你就永远无法保证安全。
- ……**你熟悉的**发行版。如果你不了解 Linux，我建议先试用一段时间，再尝试加固它。你应当能够熟练使用并了解基本操作，例如如何安装软件、配置文件在哪里等……
- ……**拥有良好支持的**发行版。即便最有经验的管理员也偶尔需要帮助。知道去哪里求助，能让你少受很多折磨。

([目录](#table-of-contents))

<a id="installing-linux"></a>
### 安装 Linux

安装 Linux 超出了本指南的范围，因为每个发行版的安装方式都不同，而且通常已有完善的安装说明。如果需要帮助，请先查阅所用发行版的文档。无论选择哪种发行版，整体流程通常如下：

1. 下载 ISO
1. 将其刻录/复制/传输到安装介质（例如 CD 或 U 盘）
1. 从安装介质启动服务器
1. 按提示完成安装

如果有专家安装选项，请使用该选项，以便更严格地控制服务器上运行的内容。**只安装绝对必要的组件。**就我个人而言，除了 SSH，我不会安装任何其他内容。另外，请勾选磁盘加密选项。

([目录](#table-of-contents))

<a id="prepost-installation-requirements"></a>
### 安装前/后的要求

- 如果你要在路由器上开放端口，以便从外部访问服务器，请先禁用端口转发，直到系统启动完毕并完成加固。
- 除非你会始终在服务器旁直接操作，否则就需要远程访问，因此请确保 SSH 正常工作。
- 让系统保持最新状态（例如，在基于 Debian 的系统上运行 `sudo apt update && sudo apt upgrade`）。
- 务必完成与你的具体设置有关的所有任务，例如：
  - 配置网络
  - 在 `/etc/fstab` 中配置挂载点
  - 创建初始用户账户
  - 安装所需的核心软件，例如 `man`
  - 等等……
- 你的服务器需要能够发送电子邮件，以便你接收重要的安全提醒。如果不打算设置邮件服务器，请查看[使用隐式 TLS 将 Gmail 和 Exim4 配置为 MTA](#gmail-and-exim4-as-mta-with-implicit-tls)。
- 我还建议在开始使用本指南之前**通读** [CIS 基准](https://www.cisecurity.org/cis-benchmarks/)，理解其中的内容。我的建议是先通读当前这份指南，**然后**再阅读 CIS 指南；这样，只要两者存在差异，就以 CIS 的建议为准。

([目录](#table-of-contents))

<a id="other-important-notes"></a>
### 其他重要注意事项

- 本指南在 Debian 上编写和测试。下面的大多数内容应该也适用于其他发行版。如果发现某项内容不适用，请[联系我](#contacting-me)。各发行版之间最主要的差异在于软件包管理系统。由于我使用 Debian，因此会提供适用于所有[基于 Debian 的发行版](https://www.debian.org/derivatives/)的相应 `apt` 命令。如果有人愿意[提供](#contributing)其他发行版的相应命令，我会将其加入本指南。
- 文件路径和设置也可能略有不同——如果遇到问题，请查阅所用发行版的文档。
- 开始之前请通读整份指南。你的使用场景和/或原则可能要求跳过某项操作，或改变操作顺序。
- 不要在不理解内容的情况下**盲目**复制粘贴。有些命令必须根据你的需求修改后才能使用，例如用户名。

([目录](#table-of-contents))

<a id="using-ansible-playbooks-to-secure-your-linux-server"></a>
### 使用 Ansible playbook 加固你的 Linux 服务器
本指南的 Ansible playbook 可从[使用 Ansible 加固 Linux 服务器](https://github.com/moltenbit/How-To-Secure-A-Linux-Server-With-Ansible)获取。

务必根据需要编辑变量，并事先阅读所有任务，确认它们不会破坏你的系统。运行 playbook 后，请确保所有设置均按你的需要进行了配置！

1. 安装 [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
2. git clone [使用 Ansible 加固 Linux 服务器](https://github.com/moltenbit/How-To-Secure-A-Linux-Server-With-Ansible)
3. [创建 SSH 公钥/私钥](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server#ssh-publicprivate-keys)
  ```
  ssh-keygen -t ed25519
  ```
   
5. 根据需要更改 *group_vars/variables.yml* 中的所有变量。
6. 运行 playbook 前，启用通过 SSH 以 root 身份登录：
   
  ```
  nano /etc/ssh/sshd_config
  [...]
  PermitRootLogin yes
  [...]
  ```

7. 建议：为系统配置静态 IP 地址。
8. 将系统的 IP 地址添加到 *hosts.yml*。

&nbsp;

使用安装服务器时指定的 root 密码运行 requirements playbook：

    ansible-playbook --inventory hosts.yml --ask-pass requirements-playbook.yml

&nbsp;

使用你在 *variables.yml* 文件中指定的新用户密码运行主 playbook：

    ansible-playbook --inventory hosts.yml --ask-pass main-playbook.yml

&nbsp;

如果需要多次运行 playbook，请记得使用 SSH 密钥和新的 SSH 端口：

    ansible-playbook --inventory hosts.yml -e ansible_ssh_port=SSH_PORT --key-file /PATH/TO/SSH/KEY main-playbook.yml


([目录](#table-of-contents))

<a id="the-ssh-server"></a>
## SSH 服务器

<a id="important-note-before-you-make-ssh-changes"></a>
### 修改 SSH 前的重要说明

强烈建议在**更改并应用 SSH 配置之前**，另开一个连接到服务器的终端。这样，即使你把自己挡在第一个终端会话之外，仍有一个已连接的会话可用于修复问题。

感谢 [Sonnenbrand](https://github.com/Sonnenbrand) 提供这个[建议](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/issues/56)。

<a id="ssh-publicprivate-keys"></a>
### SSH 公钥/私钥

#### 原因

使用 SSH 公钥/私钥比使用密码更安全。由于无需输入密码，连接服务器也会更方便、更快捷。

#### 工作原理

更多细节请查看下面的参考资料。从宏观上说，公钥/私钥通过一对密钥来验证身份。

1. 其中一个是**公钥**，**只能加密数据**，不能解密
1. 另一个是**私钥**，可以解密数据

使用 SSH 时，公钥和私钥都在客户端创建。你需要妥善保护这两个密钥，尤其是私钥。虽然公钥本来就是要公开的，但确保两个密钥都不落入坏人之手仍是明智之举。

连接 SSH 服务器时，SSH 会在目标服务器的 `~/.ssh/authorized_keys` 文件中查找与发起连接的客户端相匹配的公钥。请注意，该文件位于你尝试登录的账户的**主目录**中。因此，创建公钥后，需要将其追加到 `~/.ssh/authorized_keys`。一种做法是把公钥复制到 U 盘，再通过物理方式传输到服务器；另一种做法是使用 [`ssh-copy-id`](https://www.ssh.com/ssh/copy-id) 传输并追加公钥。

创建密钥并将公钥追加到主机上的 `~/.ssh/authorized_keys` 后，SSH 会使用公钥和私钥验证身份，然后建立安全连接。身份验证过程相当复杂，但 [Digital Ocean](https://www.digitalocean.com/community/tutorials/understanding-the-ssh-encryption-and-connection-process) 对其工作方式作了很好的说明。从宏观上说，服务器会使用公钥加密一条质询消息，再将其发送给客户端，以此验证身份。如果客户端无法使用私钥解密质询消息，身份就无法得到验证，也不会建立连接。

公钥/私钥之所以被认为更安全，是因为建立 SSH 连接必须拥有私钥。如果你设置了 [`PasswordAuthentication no`（位于 `/etc/ssh/sshd_config` 中）](#PasswordAuthentication)，SSH 就不允许没有私钥的客户端连接。

你还可以为密钥设置口令；使用公钥/私钥连接时，就必须输入该密钥口令。请注意，这样做意味着无法将该密钥用于自动化，因为脚本无法提供口令。许多 Linux 发行版都附带 `ssh-agent`（而且它通常已经在运行），它可以在一段可配置的时间内将未加密的私钥保存在内存中。只需运行 `ssh-add`，它就会提示你输入口令。在设定的有效时间过去之前，系统不会再次要求输入口令。

我们将使用 Ed25519 密钥；据 [https://linux-audit.com/](https://linux-audit.com/using-ed25519-openssh-keys-instead-of-dsa-rsa-ecdsa/) 所述：

> 它使用椭圆曲线签名方案，比 ECDSA 和 DSA 提供更好的安全性，同时还具有良好的性能。

#### 目标

- Ed25519 SSH 公钥/私钥：
  - 私钥位于客户端
  - 公钥位于服务器

#### 注意事项

- 对于每一台将用于连接服务器的计算机，以及每一个将用于登录服务器的账户，都需要执行此步骤。

#### 参考资料

- https://www.ssh.com/ssh/public-key-authentication
- https://help.ubuntu.com/community/SSH/OpenSSH/Keys
- https://linux-audit.com/using-ed25519-openssh-keys-instead-of-dsa-rsa-ecdsa/
- https://www.digitalocean.com/community/tutorials/understanding-the-ssh-encryption-and-connection-process
- https://wiki.archlinux.org/index.php/SSH_Keys
- https://www.ssh.com/ssh/copy-id
- `man ssh-keygen`
- `man ssh-copy-id`
- `man ssh-add`

#### 操作步骤

1. 在将用于连接服务器的计算机（也就是**客户端**，不是服务器本身）上，使用 `ssh-keygen` 创建 [Ed25519](https://linux-audit.com/using-ed25519-openssh-keys-instead-of-dsa-rsa-ecdsa/) 密钥：

    ``` bash
    ssh-keygen -t ed25519
    ```

    > ```
    > Generating public/private ed25519 key pair.
    > Enter file in which to save the key (/home/user/.ssh/id_ed25519):
    > Created directory '/home/user/.ssh'.
    > Enter passphrase (empty for no passphrase):
    > Enter same passphrase again:
    > Your identification has been saved in /home/user/.ssh/id_ed25519.
    > Your public key has been saved in /home/user/.ssh/id_ed25519.pub.
    > The key fingerprint is:
    > SHA256:F44D4dr2zoHqgj0i2iVIHQ32uk/Lx4P+raayEAQjlcs user@client
    > The key's randomart image is:
    > +--[ED25519 256]--+
    > |xxxx  x          |
    > |o.o +. .         |
    > | o o oo   .      |
    > |. E oo . o .     |
    > | o o. o S o      |
    > |... .. o o       |
    > |.+....+ o        |
    > |+.=++o.B..       |
    > |+..=**=o=.       |
    > +----[SHA256]-----+
    > ```

    **注意**：如果设置了口令，那么每次使用此密钥连接服务器时都需要输入口令，除非你正在使用 `ssh-agent`。

1. 现在需要把客户端上的公钥 `~/.ssh/id_ed25519.pub` **追加**到服务器的 `~/.ssh/authorized_keys` 文件中。由于此时我们大概仍在家中的 LAN 内，应该不会遭受 [MIM](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) 攻击，因此将使用 `ssh-copy-id` 传输并追加公钥：

    ``` bash
    ssh-copy-id user@server
    ```

    > ```
    > /usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/user/.ssh/id_ed25519.pub"
    > The authenticity of host 'host (192.168.1.96)' can't be established.
    > ECDSA key fingerprint is SHA256:QaDQb/X0XyVlogh87sDXE7MR8YIK7ko4wS5hXjRySJE.
    > Are you sure you want to continue connecting (yes/no)? yes
    > /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
    > /usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
    > user@host's password:
    > 
    > Number of key(s) added: 1
    > 
    > Now try logging into the machine, with:   "ssh 'user@host'"
    > and check to make sure that only the key(s) you wanted were added.
    > ```

现在正适合[执行与你的具体设置有关的任务](#prepost-installation-requirements)。

([目录](#table-of-contents))

<a id="create-ssh-group-for-allowgroups"></a>
### 为 AllowGroups 创建 SSH 组

#### 原因

这样可以方便地控制哪些人能够通过 SSH 连接服务器。使用一个组后，我们可以快速向组中添加账户或从组中移除账户，从而迅速允许或禁止相应账户通过 SSH 访问服务器。

#### 工作原理

我们将使用 [AllowGroups 选项](#AllowGroups)，它位于 SSH 配置文件 [`/etc/ssh/sshd_config`](#secure-etcsshsshd_config) 中。该选项会让 SSH 服务器只允许某个特定 UNIX 组的成员通过 SSH 登录；不在该组中的任何人都无法通过 SSH 登录。

#### 目标

- 创建一个 UNIX 组，供[加固 `/etc/ssh/sshd_config`](#secure-etcsshsshd_config)时用于限制哪些人能够通过 SSH 连接服务器

#### 注意事项

- 这是支持 `AllowGroup` 配置的前置步骤，该配置会在[加固 `/etc/ssh/sshd_config`](#secure-etcsshsshd_config)时设置。

#### 参考资料

- `man groupadd`
- `man usermod`

#### 操作步骤

1. 创建一个组：

    ``` bash
    sudo groupadd sshusers
    ```

1. 将账户添加到该组：

    ``` bash
    sudo usermod -a -G sshusers user1
    sudo usermod -a -G sshusers user2
    sudo usermod -a -G sshusers ...
    ```

    服务器上每个需要 SSH 访问权限的账户都必须执行此操作。

([目录](#table-of-contents))

<a id="secure-etcsshsshd_config"></a>
### 加固 `/etc/ssh/sshd_config`

#### 原因

SSH 是进入服务器的一扇门。如果你在路由器上开放端口，以便从家庭网络之外通过 SSH 连接服务器，这一点尤其重要。如果没有妥善加固，恶意行为者可能利用 SSH 未经授权访问你的系统。

#### 工作原理

`/etc/ssh/sshd_config` 是 SSH 服务器使用的默认配置文件。我们将通过此文件指定 SSH 服务器应使用的选项。

#### 目标

- 安全的 SSH 配置

#### 注意事项

- 请确保先完成[为 AllowGroups 创建 SSH 组](#create-ssh-group-for-allowgroups)。

#### 参考资料

- Mozilla 面向 OpenSSH 6.7+ 的 OpenSSH 指南：https://infosec.mozilla.org/guidelines/openssh#modern-openssh-67
- https://linux-audit.com/audit-and-harden-your-ssh-configuration/
- https://www.ssh.com/ssh/sshd_config/
- https://www.techbrown.com/harden-ssh-secure-linux-vps-server/ （链接已失效；请尝试 http://web.archive.org/web/20200413100933/https://www.techbrown.com/harden-ssh-secure-linux-vps-server/ ）
- https://serverfault.com/questions/660160/openssh-difference-between-internal-sftp-and-sftp-server/660325
- `man sshd_config`
- 感谢 [than0s](https://github.com/than0s) 提供[查找重复设置的方法](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/issues/38)。

#### 操作步骤

1. 备份 OpenSSH 服务器配置文件 `/etc/ssh/sshd_config`，并删除注释以便阅读：

    ``` bash
    sudo cp --archive /etc/ssh/sshd_config /etc/ssh/sshd_config-COPY-$(date +"%Y%m%d%H%M%S")
    sudo sed -i -r -e '/^#|^$/ d' /etc/ssh/sshd_config
    ```

1. 编辑 `/etc/ssh/sshd_config`，然后查找并修改或添加以下设置；无论你的配置/环境如何，都应应用这些设置：

    **注意**：SSH 不接受相互矛盾的重复设置。例如，如果先有 `KbdInteractiveAuthentication no`，后有 `KbdInteractiveAuthentication yes`，SSH 会采用第一项并忽略第二项。你的 `/etc/ssh/sshd_config` 文件中可能已经包含下面的某些设置/行。为了避免问题，需要手动检查 `/etc/ssh/sshd_config` 文件，并处理所有相互矛盾的重复设置。

	**注意：**如果运行 OpenSSH 9.1 或更高版本，请取消下面配置中 `RequiredRSASize 3072` 行的注释。这样会强制要求 RSA 密钥至少为 3072 位，并在身份验证时拒绝更短的 RSA 密钥。此设置只影响 RSA 密钥；使用 ED25519 或 ECDSA 密钥不会受到影响。你可以使用 `ssh-keygen -l -f ~/.ssh/id_rsa` 检查密钥类型和长度。在较早的 OpenSSH 版本上，请让此行保持注释状态，否则 sshd 将无法启动。
   
    ```
    ########################################################################################################
    # start settings from https://infosec.mozilla.org/guidelines/openssh#modern-openssh-67 as of 2019-01-01
    ########################################################################################################

    # Supported HostKey algorithms by order of preference.
    HostKey /etc/ssh/ssh_host_ed25519_key
    HostKey /etc/ssh/ssh_host_rsa_key
    HostKey /etc/ssh/ssh_host_ecdsa_key

    KexAlgorithms curve25519-sha256@libssh.org,ecdh-sha2-nistp521,ecdh-sha2-nistp384,ecdh-sha2-nistp256,diffie-hellman-group-exchange-sha256

    Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr

    MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-sha2-512,hmac-sha2-256,umac-128@openssh.com

    # LogLevel VERBOSE logs user's key fingerprint on login. Needed to have a clear audit track of which key was using to log in.
    LogLevel VERBOSE

    # Use kernel sandbox mechanisms where possible in unprivileged processes
    # Systrace on OpenBSD, Seccomp on Linux, seatbelt on MacOSX/Darwin, rlimit elsewhere.
    # Note: This setting is deprecated in OpenSSH 7.5 (https://www.openssh.com/txt/release-7.5)
    # UsePrivilegeSeparation sandbox

    ########################################################################################################
    # end settings from https://infosec.mozilla.org/guidelines/openssh#modern-openssh-67 as of 2019-01-01
    ########################################################################################################

    # don't let users set environment variables
    PermitUserEnvironment no

    # Log sftp level file access (read/write/etc.) that would not be easily logged otherwise.
    Subsystem sftp  internal-sftp -f AUTHPRIV -l INFO

    # disable X11 forwarding as X11 is very insecure
    # you really shouldn't be running X on a server anyway
    X11Forwarding no

    # disable port forwarding
    AllowTcpForwarding no
    AllowStreamLocalForwarding no
    GatewayPorts no
    PermitTunnel no

    # don't allow login if the account has an empty password
    PermitEmptyPasswords no

    # ignore .rhosts and .shosts
    IgnoreRhosts yes

    # verify hostname matches IP
    UseDNS yes

    Compression no
    
    # TCP keepalive is spoofable (runs outside the encrypted channel)
	# Use ClientAlive instead (runs inside the encrypted channel)
    TCPKeepAlive no
    
    AllowAgentForwarding no
    PermitRootLogin no

    # don't allow .rhosts or /etc/hosts.equiv
    HostbasedAuthentication no

    # OpenSSH 9.1 and later
    # Enforce a minimum RSA key size of 3072 bits
    # https://www.keylength.com/en/compare/
    # RequiredRSASize 3072

    ```

1. 然后**查找并修改或添加**以下设置，并根据你的需求设定值：

    |设置|有效值|示例|说明|注意事项|
    |--|--|--|--|--|
    |<a name="AllowGroups"></a>**AllowGroups**|本地 UNIX 组名|`AllowGroups sshusers`|允许通过 SSH 访问的组||
    |**ClientAliveCountMax**|数值|`ClientAliveCountMax 3`|在未收到响应时，最多发送的客户端存活消息数||
    |**ClientAliveInterval**|秒数|`ClientAliveInterval 15`|请求响应前等待的秒数||
    |**ListenAddress**|以空格分隔的本地地址列表|<ul><li>`ListenAddress 0.0.0.0`</li><li>`ListenAddress 192.168.1.100`</li></ul>|`sshd` 应监听的本地地址|重要细节请参阅[问题 #1](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/issues/1)。|
    |**LoginGraceTime**|秒数|`LoginGraceTime 30`|登录超时前等待的秒数||
    |**MaxAuthTries**|数值|`MaxAuthTries 2`|允许的最大登录尝试次数||
    |**MaxSessions**|数值|`MaxSessions 2`|最多可打开的会话数||
    |**MaxStartups**|数值或 `start:rate:full`|`MaxStartups 10:30:60`|未经身份验证的 SSH 连接最大数量||
    |<a name="PasswordAuthentication"></a>**PasswordAuthentication**|`yes` 或 `no`|`PasswordAuthentication no`|是否允许使用密码登录||
    |**Port**|任意已开放/可用的端口号|`Port 22`|`sshd` 应监听的端口||

    有关这些设置含义的更多信息，请查看 `man sshd_config`。

1. 确保不存在相互矛盾的重复设置。下面的命令不应产生任何输出。

    ```bash
    awk 'NF && $1!~/^(#|HostKey)/{print $1}' /etc/ssh/sshd_config | sort | uniq -c | grep -v ' 1 '
    ```

1. 重启前验证 OpenSSH 服务器配置：

    ``` bash
    sudo sshd -t
    ```

1. 重启 ssh：

    ``` bash
    sudo service ssh restart
    ```

1. 可以使用 `sshd -T` 检查配置是否生效，并核对其输出：

    ``` bash
    sudo sshd -T
    ```

    > ```
    > port 22
    > addressfamily any
    > listenaddress [::]:22
    > listenaddress 0.0.0.0:22
    > usepam yes
    > logingracetime 30
    > x11displayoffset 10
    > maxauthtries 2
    > maxsessions 2
    > clientaliveinterval 15
    > clientalivecountmax 3
    > streamlocalbindmask 0177
    > permitrootlogin no
    > ignorerhosts yes
    > ignoreuserknownhosts no
    > hostbasedauthentication no
    > ...
    > subsystem sftp internal-sftp -f AUTHPRIV -l INFO
    > maxstartups 2:30:2
    > permittunnel no
    > ipqos lowdelay throughput
    > rekeylimit 0 0
    > permitopen any
    > ```

([目录](#table-of-contents))

<a id="remove-short-diffie-hellman-keys"></a>
### 移除短 Diffie-Hellman 密钥

#### 原因

根据 [Mozilla 面向 OpenSSH 6.7+ 的 OpenSSH 指南](https://infosec.mozilla.org/guidelines/openssh#modern-openssh-67)，“所有正在使用的 Diffie-Hellman 模数都应至少达到 3072 位”。

SSH 使用 Diffie-Hellman 算法建立安全连接。模数（密钥长度）越大，加密就越强。

#### 目标

- 移除所有长度小于 3072 位的 Diffie-Hellman 密钥

#### 参考资料

- Mozilla 面向 OpenSSH 6.7+ 的 OpenSSH 指南：https://infosec.mozilla.org/guidelines/openssh#modern-openssh-67
- https://infosec.mozilla.org/guidelines/key_management
- `man moduli`

#### 操作步骤

1. 备份 SSH 的模数文件 `/etc/ssh/moduli`：

    ``` bash
    sudo cp --archive /etc/ssh/moduli /etc/ssh/moduli-COPY-$(date +"%Y%m%d%H%M%S")
    ```

1. 移除过短的模数：

    ``` bash
    sudo awk '$5 >= 3071' /etc/ssh/moduli | sudo tee /etc/ssh/moduli.tmp
    sudo mv /etc/ssh/moduli.tmp /etc/ssh/moduli
    ````

([目录](#table-of-contents))

<a id="2famfa-for-ssh"></a>
### 为 SSH 配置 2FA/MFA

#### 原因

虽然 SSH 能像一名出色的保安一样守护你的门窗，但它仍是一扇可见的门，恶意行为者能够看到它，并尝试通过暴力破解闯入。[Fail2ban](#application-intrusion-detection-and-prevention-with-fail2ban) 会监控这些暴力破解尝试，但安全防护永远不嫌多。要求提供两个身份验证因素，可以增加一层安全保障。

使用双因素身份验证（2FA）/多因素身份验证（MFA）时，任何想进入的人都必须拥有**两把**钥匙，这会增加恶意行为者入侵的难度。这两把钥匙是：

1. 他们的密码
1. 每 30 秒变化一次的 6 位令牌

缺少其中任何一把钥匙，他们都无法进入。

#### 不采用的理由

许多人可能会觉得这种使用体验繁琐或恼人。此外，能否访问系统还取决于用于生成验证码的配套身份验证器应用。

#### 工作原理

在 Linux 上，PAM 负责身份验证。PAM 有四类任务，可在 https://en.wikipedia.org/wiki/Linux_PAM 了解相关信息。本节讨论其中的身份验证任务。

登录服务器时，无论是直接从控制台登录还是通过 SSH 登录，你所经过的那扇“门”都会把请求发送给 PAM 的身份验证任务，PAM 随后会要求提供并验证密码。你可以自定义每扇门使用的规则。例如，直接从控制台登录时可以使用一套规则，通过 SSH 登录时则使用另一套规则。

本节将更改通过 SSH 登录时使用的身份验证规则，要求同时提供密码和 6 位验证码。

我们将使用 Google 的 libpam-google-authenticator PAM 模块创建并验证 [TOTP](https://en.wikipedia.org/wiki/Time-based_One-time_Password_algorithm) 密钥。https://fastmail.blog/2016/07/22/how-totp-authenticator-apps-work/ 和 https://jemurai.com/2018/10/11/how-it-works-totp-based-mfa/ 对 TOTP 的工作原理作了很好的介绍。

我们要做的是配置服务器的 SSH PAM，让它先要求用户输入密码，再输入数字令牌。PAM 会验证用户密码；如果密码正确，就把身份验证请求转交给 libpam-google-authenticator，由后者要求提供并验证 6 位令牌。只有当所有验证都正确无误时，身份验证才会成功，并允许用户登录。

#### 目标

- 为所有 SSH 连接启用 2FA/MFA

#### 注意事项

- 执行这些操作之前，你应对 2FA/MFA 的工作原理有所了解，而且需要在手机上安装身份验证器应用才能继续。
- 我们将使用 [google-authenticator-libpam](https://github.com/google/google-authenticator-libpam)。
- 采用下面的配置后，用户只有在使用密码登录时才需要输入 2FA/MFA 验证码；使用 [SSH 公钥/私钥](#ssh-publicprivate-keys)时则**不需要**。如果希望公钥身份验证也要求提供 TOTP 验证码，请配置 `AuthenticationMethods`，要求 `keyboard-interactive` 位于 `publickey` 之后。
- `nullok` 选项允许没有 `~/.google_authenticator` 文件的用户在不提供验证码的情况下登录。如果每个账户都必须强制使用 MFA，请在所有预期的 SSH 用户完成注册后移除 `nullok`。

#### 参考资料

- https://github.com/google/google-authenticator-libpam
- https://en.wikipedia.org/wiki/Linux_PAM
- https://en.wikipedia.org/wiki/Time-based_One-time_Password_algorithm
- https://fastmail.blog/2016/07/22/how-totp-authenticator-apps-work/
- https://jemurai.com/2018/10/11/how-it-works-totp-based-mfa/

#### 操作步骤

1. 安装 libpam-google-authenticator。

    在基于 Debian 的系统上：

    ``` bash
    sudo apt install libpam-google-authenticator
    ```

1. **确保当前登录的就是要启用 2FA/MFA 的账户**，然后**执行** `google-authenticator` 创建所需的令牌数据：

    ``` bash
    google-authenticator
    ```

    > ```
    > Do you want authentication tokens to be time-based (y/n) y
    > https://www.google.com/chart?chs=200x200&chld=M|0&cht=qr&chl=otpauth://totp/user@host%3Fsecret%3DR4ZWX34FQKZROVX7AGLJ64684Y%26issuer%3Dhost
    > 
    > ...
    > 
    > Your new secret key is: R3NVX3FFQKZROVX7AGLJUGGESY
    > Your verification code is 751419
    > Your emergency scratch codes are:
    >   12345678
    >   90123456
    >   78901234
    >   56789012
    >   34567890
    > 
    > Do you want me to update your "/home/user/.google_authenticator" file (y/n) y
    > 
    > Do you want to disallow multiple uses of the same authentication
    > token? This restricts you to one login about every 30s, but it increases
    > your chances to notice or even prevent man-in-the-middle attacks (y/n) Do you want to disallow multiple uses of the same authentication
    > token? This restricts you to one login about every 30s, but it increases
    > your chances to notice or even prevent man-in-the-middle attacks (y/n) y
    > 
    > By default, tokens are good for 30 seconds. In order to compensate for
    > possible time-skew between the client and the server, we allow an extra
    > token before and after the current time. If you experience problems with
    > poor time synchronization, you can increase the window from its default
    > size of +-1min (window size of 3) to about +-4min (window size of
    > 17 acceptable tokens).
    > Do you want to do so? (y/n) y
    > 
    > If the computer that you are logging into isn't hardened against brute-force
    > login attempts, you can enable rate-limiting for the authentication module.
    > By default, this limits attackers to no more than 3 login attempts every 30s.
    > Do you want to enable rate-limiting (y/n) y
    > ```

    请注意，此处执行该命令时**不使用 root 身份**。

    对它提出的所有问题选择默认选项（多数情况下为 y），并记得保存紧急备用码。

1. 备份 PAM 的 SSH 配置文件 `/etc/pam.d/sshd`：

    ``` bash
    sudo cp --archive /etc/pam.d/sshd /etc/pam.d/sshd-COPY-$(date +"%Y%m%d%H%M%S")
    ```

1. 现在需要把它启用为 SSH 身份验证方法。将以下一行添加到 `/etc/pam.d/sshd`：

    ```
    auth       required     pam_google_authenticator.so nullok
    ```

    **注意**：有关 `nullok` 的含义，请查看[这里](https://github.com/google/google-authenticator-libpam/blob/master/README.md#nullok)。

    [给想省事的人](#editing-configuration-files---for-the-lazy)：

    ``` bash
    echo -e "\nauth       required     pam_google_authenticator.so nullok         # added by $(whoami) on $(date +"%Y-%m-%d @ %H:%M:%S")" | sudo tee -a /etc/pam.d/sshd
    ```

1. 在 `/etc/ssh/sshd_config` 中添加或编辑以下一行，让 SSH 使用它：

    ```
    KbdInteractiveAuthentication yes
    ```

    [给想省事的人](#editing-configuration-files---for-the-lazy)：

    ``` bash
    sudo sed -i -r -e "s/^(kbdinteractiveauthentication|challengeresponseauthentication)( .*)$/# \1\2         # commented by $(whoami) on $(date +"%Y-%m-%d @ %H:%M:%S")/I" /etc/ssh/sshd_config
    echo -e "\nKbdInteractiveAuthentication yes         # added by $(whoami) on $(date +"%Y-%m-%d @ %H:%M:%S")" | sudo tee -a /etc/ssh/sshd_config
    ```

1. 重启前验证 OpenSSH 服务器配置：

    ``` bash
    sudo sshd -t
    ```

1. 重启 ssh：

    ``` bash
    sudo service ssh restart
    ```

([目录](#table-of-contents))

<a id="the-basics"></a>
## 基础设置

<a id="limit-who-can-use-sudo"></a>
### 限制 sudo 的使用者

#### 原因

sudo 允许账户以其他账户（包括 **root**）的身份运行命令。我们要确保只有指定的账户能够使用 sudo。

#### 目标

- 仅将 sudo 权限授予指定组的成员

#### 注意事项

- 安装程序可能已经完成此项配置，或者系统中可能已有一个为此用途准备的专用组，因此请先检查。
  - Debian 会创建 sudo 组。要查看该组中的用户（也就是拥有 sudo 权限的用户）：
	  
	  ```
	  cat /etc/group | grep "sudo"
	  ```
  - RedHat 会创建 wheel 组
- 有些发行版将 `sudo` 配置为不要求密码，相关说明请参阅 [https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/issues/39](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/issues/39)。感谢 [sbrl](https://github.com/sbrl) 分享此信息。

#### 操作步骤

1. 创建一个组：

    ``` bash
    sudo groupadd sudousers
    ```

1. 将账户添加到该组：

    ``` bash
    sudo usermod -a -G sudousers user1
    sudo usermod -a -G sudousers user2
    sudo usermod -a -G sudousers  ...
    ```

    服务器上每个需要 sudo 权限的账户都必须执行此操作。

1. 备份 sudo 的配置文件 `/etc/sudoers`：

    ``` bash
    sudo cp --archive /etc/sudoers /etc/sudoers-COPY-$(date +"%Y%m%d%H%M%S")
    ```

1. 编辑 sudo 的配置文件 `/etc/sudoers`：

    ``` bash
    sudo visudo
    ```

1. 如果下面一行尚不存在，请将其添加到文件中，让 sudo 只允许 `sudousers` 组中的用户使用 sudo：

    ```
    %sudousers   ALL=(ALL:ALL) ALL
    ```

([目录](#table-of-contents))

<a id="limit-who-can-use-su"></a>
### 限制 su 的使用者

#### 原因

su 同样允许账户以其他账户（包括 **root**）的身份运行命令。我们要确保只有指定的账户能够使用 su。

#### 目标

- 仅将 su 权限授予指定组的成员

#### 参考资料

- 感谢 [olavim](https://github.com/olavim) 分享[这个想法](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/issues/41)

#### 操作步骤

1. 创建一个组：

    ``` bash
    sudo groupadd suusers
    ```

1. 将账户添加到该组：

    ``` bash
    sudo usermod -a -G suusers user1
    sudo usermod -a -G suusers user2
    sudo usermod -a -G suusers  ...
    ```

    服务器上每个需要 sudo 权限的账户都必须执行此操作。

1. 将 `/bin/su` 配置为只有该组中的用户才能执行：

    ``` bash
    sudo dpkg-statoverride --update --add root suusers 4750 /bin/su
    ```

([目录](#table-of-contents))

<a id="run-applications-in-a-sandbox-with-firejail"></a>
### 使用 FireJail 在沙箱中运行应用程序

#### 原因

对于许多应用程序来说，在沙箱中运行无疑更好。

强烈建议对浏览器（尤其是闭源浏览器）和电子邮件客户端这样做。

#### 目标

- 将应用程序限制在 jail（少数几个安全目录）中，并阻止其访问系统的其他部分

#### 参考资料

- 感谢 [FireJail](https://firejail.wordpress.com/)

#### 操作步骤

1. 安装该软件：

    ``` bash
    sudo apt install firejail firejail-profiles
    ```
    
    注意：对于 Debian 10 Stable，建议使用官方 Backport：

    ``` bash
    sudo apt install -t buster-backports firejail firejail-profiles
    ```

2. 让安装在 `/usr/bin` 或 `/bin` 中的应用程序只能在沙箱中运行（下面给出了几个示例）：

    ``` bash
    sudo ln -s /usr/bin/firejail /usr/local/bin/google-chrome-stable
    sudo ln -s /usr/bin/firejail /usr/local/bin/firefox
    sudo ln -s /usr/bin/firejail /usr/local/bin/chromium
    sudo ln -s /usr/bin/firejail /usr/local/bin/evolution
    sudo ln -s /usr/bin/firejail /usr/local/bin/thunderbird
    ```

3. 像往常一样运行应用程序（通过终端或启动器），然后检查它是否正在 jail 中运行：

    ``` bash
    firejail --list
    ```

4. 让沙箱化的应用程序恢复为以前的运行方式（示例：firefox）

    ``` bash
    sudo rm /usr/local/bin/firefox
    ```

([目录](#table-of-contents))

<a id="ntp-client"></a>
### NTP 客户端

#### 原因

许多安全协议都依赖时间。如果系统时间不正确，可能会对服务器造成负面影响。NTP 客户端可以让系统时间与[全球 NTP 服务器](https://en.wikipedia.org/wiki/Network_Time_Protocol)保持同步，从而解决这个问题。

#### 工作原理

NTP 是 Network Time Protocol（网络时间协议）的缩写。在本指南的场景中，服务器上的 NTP 客户端会从官方服务器获取标准时间，并用它更新服务器时间。所有公共 NTP 服务器请查看 https://www.pool.ntp.org/en/ 。
> **注意：**从 **Debian 13 (Trixie)** 开始，传统的 `ntp` 软件包已被移除。运行 `sudo apt install ntp` 会失败，并显示 *"Package ntp has no installation candidate"*。由于本指南仅将 NTP 用作**客户端**（用于同步服务器时钟），Debian 13+ 上的推荐做法是使用已预装且无需额外软件包的 `systemd-timesyncd`。请参阅下面的 [Debian 13+ 操作步骤](#debian-13-trixie-and-later-systemd-timesyncd)。

#### 目标

- 安装 NTP 客户端，并让服务器时间保持同步

#### 参考资料

- https://cloudpro.zone/index.php/2018/01/27/debian-9-3-server-setup-guide-part-4/
- https://en.wikipedia.org/wiki/Network_Time_Protocol
- https://www.pool.ntp.org/en/
- https://serverfault.com/questions/957302/securing-hardening-ntp-client-on-linux-servers-config-file/957450#957450
- https://tf.nist.gov/tf-cgi/servers.cgi

#### 操作步骤

<a id="debian-13-trixie-and-later-systemd-timesyncd"></a>
##### Debian 13 (Trixie) 及更高版本：systemd-timesyncd

`systemd-timesyncd` 是 Debian 已自带的轻量级 SNTP 客户端。与功能完整的 `ntpd` 守护进程不同，它不监听任何端口，因此攻击面更小。对于本指南的用途——让服务器时钟保持同步——它已经足够。

1. 启用 NTP 同步：

    ``` bash
    sudo timedatectl set-ntp true
    ```

1. 验证它是否正常工作：

    ``` bash
    timedatectl status
    ```

    输出中应出现 `NTP service: active` 和 `System clock synchronized: yes`。

1. 配置可信的 NTP 服务器。先备份配置文件，然后进行编辑：

    ``` bash
    sudo cp --archive /etc/systemd/timesyncd.conf /etc/systemd/timesyncd.conf-COPY-$(date +"%Y%m%d%H%M%S")
    ```

    编辑 `/etc/systemd/timesyncd.conf`，取消 `[Time]` 部分的注释并进行设置：

    ```
    [Time]
    NTP=pool.ntp.org
    FallbackNTP=0.debian.pool.ntp.org 1.debian.pool.ntp.org 2.debian.pool.ntp.org
    ```

    [给想省事的人](#editing-configuration-files---for-the-lazy)：

    ``` bash
    sudo sed -i -r -e "s/^#?NTP=.*$/NTP=pool.ntp.org         # added by $(whoami) on $(date +"%Y-%m-%d @ %H:%M:%S")/" /etc/systemd/timesyncd.conf
    sudo sed -i -r -e "s/^#?FallbackNTP=.*$/FallbackNTP=0.debian.pool.ntp.org 1.debian.pool.ntp.org 2.debian.pool.ntp.org         # added by $(whoami) on $(date +"%Y-%m-%d @ %H:%M:%S")/" /etc/systemd/timesyncd.conf
    ```

1. 重启服务以应用更改：

    ``` bash
    sudo systemctl restart systemd-timesyncd
    ```

1. 检查同步状态：

    ``` bash
    timedatectl timesync-status
    ```

    > ```
    >        Server: 108.61.56.35 (pool.ntp.org)
    > Poll interval: 32s (min: 32s; max: 34min 8s)
    >          Leap: normal
    >       Version: 4
    >       Stratum: 2
    >     Reference: C342F10A
    >     Precision: 1us (2^0)
    >  Root distance: 24.054ms (max: 5s)
    >        Offset: +2.156ms
    >         Delay: 48.567ms
    >        Jitter: 1.452ms
    >  Packet count: 3
    > ```

##### Debian 12 (Bookworm) 及更早版本：ntp 软件包

> **注意：**这些步骤仅适用于 **Debian 12 及更早版本**。在 Debian 13+ 上，`ntp` 软件包已不再可用——请改用上面的 [systemd-timesyncd 操作步骤](#debian-13-trixie-and-later-systemd-timesyncd)。

1. 安装 ntp。

    在基于 Debian 的系统上：

    ``` bash
    sudo apt install ntp
    ```
    
1. 备份 NTP 客户端配置文件 `/etc/ntp.conf`：

    ``` bash
    sudo cp --archive /etc/ntp.conf /etc/ntp.conf-COPY-$(date +"%Y%m%d%H%M%S")
    ```

1. 至少在 Debian 上，默认配置已经相当安全。我们只需确保使用 `pool` 指令，而不使用任何 `server` 指令。如果某台服务器无响应或提供错误时间，`pool` 指令允许 NTP 客户端停止使用它。为此，请注释掉所有 `server` 指令，并将下面的内容添加到 `/etc/ntp.conf`。
	
    ```
    pool pool.ntp.org iburst
    ```
    
    [给想省事的人](#editing-configuration-files---for-the-lazy)：
    
    ``` bash
    sudo sed -i -r -e "s/^((server|pool).*)/# \1         # commented by $(whoami) on $(date +"%Y-%m-%d @ %H:%M:%S")/" /etc/ntp.conf
    echo -e "\npool pool.ntp.org iburst         # added by $(whoami) on $(date +"%Y-%m-%d @ %H:%M:%S")" | sudo tee -a /etc/ntp.conf
    ```

    **`/etc/ntp.conf` 示例**：
    
    > ```
    > driftfile /var/lib/ntp/ntp.drift
    > statistics loopstats peerstats clockstats
    > filegen loopstats file loopstats type day enable
    > filegen peerstats file peerstats type day enable
    > filegen clockstats file clockstats type day enable
    > restrict -4 default kod notrap nomodify nopeer noquery limited
    > restrict -6 default kod notrap nomodify nopeer noquery limited
    > restrict 127.0.0.1
    > restrict ::1
    > restrict source notrap nomodify noquery
    > pool pool.ntp.org iburst         # added by user on 2019-03-09 @ 10:23:35
    > ```
    
1. 重启 ntp：

    ``` bash
    sudo service ntp restart
    ```

1. 检查 ntp 服务的状态：

    ``` bash
    sudo systemctl status ntp
    ```

    > ```
    > ● ntp.service - LSB: Start NTP daemon
    >    Loaded: loaded (/etc/init.d/ntp; generated; vendor preset: enabled)
    >    Active: active (running) since Sat 2019-03-09 15:19:46 EST; 4s ago
    >      Docs: man:systemd-sysv-generator(8)
    >   Process: 1016 ExecStop=/etc/init.d/ntp stop (code=exited, status=0/SUCCESS)
    >   Process: 1028 ExecStart=/etc/init.d/ntp start (code=exited, status=0/SUCCESS)
    >     Tasks: 2 (limit: 4915)
    >    CGroup: /system.slice/ntp.service
    >            └─1038 /usr/sbin/ntpd -p /var/run/ntpd.pid -g -u 108:113
    > 
    > Mar 09 15:19:46 host ntpd[1038]: Listen and drop on 0 v6wildcard [::]:123
    > Mar 09 15:19:46 host ntpd[1038]: Listen and drop on 1 v4wildcard 0.0.0.0:123
    > Mar 09 15:19:46 host ntpd[1038]: Listen normally on 2 lo 127.0.0.1:123
    > Mar 09 15:19:46 host ntpd[1038]: Listen normally on 3 enp0s3 10.10.20.96:123
    > Mar 09 15:19:46 host ntpd[1038]: Listen normally on 4 lo [::1]:123
    > Mar 09 15:19:46 host ntpd[1038]: Listen normally on 5 enp0s3 [fe80::a00:27ff:feb6:ed8e%2]:123
    > Mar 09 15:19:46 host ntpd[1038]: Listening on routing socket on fd #22 for interface updates
    > Mar 09 15:19:47 host ntpd[1038]: Soliciting pool server 108.61.56.35
    > Mar 09 15:19:48 host ntpd[1038]: Soliciting pool server 69.89.207.199
    > Mar 09 15:19:49 host ntpd[1038]: Soliciting pool server 45.79.111.114
    > ```

1. 检查 ntp 的状态：

    ``` bash
    sudo ntpq -p
    ```

    > ```
    >      remote           refid      st t when poll reach   delay   offset  jitter
    > ==============================================================================
    >  pool.ntp.org    .POOL.          16 p    -   64    0    0.000    0.000   0.000
    > *lithium.constan 198.30.92.2      2 u    -   64    1   19.900    4.894   3.951
    >  ntp2.wiktel.com 212.215.1.157    2 u    2   64    1   48.061   -0.431   0.104
    > ```

([目录](#table-of-contents))

<a id="securing-proc"></a>
### 保护 /proc

#### 原因

引用 https://linux-audit.com/linux-system-hardening-adding-hidepid-to-proc/ 的说明：

> 查看 `/proc` 时，你会发现许多文件和目录。其中很多仅以数字命名，表示有关特定进程 ID（PID）的信息。Linux 系统在默认部署状态下允许所有本地用户查看这些信息，其中包括其他用户的进程信息，而这些信息可能包含你不想与其他用户共享的敏感细节。通过调整一些文件系统配置，我们可以改变这种行为并提高系统安全性。

**注意**：这项设置可能会导致某些 `systemd` 系统出现故障。更多信息请参阅 [https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/issues/37](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/issues/37)。感谢 [nlgranger](https://github.com/nlgranger) 分享此信息。

#### 目标

- 将 `/proc` 以 `hidepid=2` 挂载，使用户只能查看自己进程的信息

#### 参考资料

- https://linux-audit.com/linux-system-hardening-adding-hidepid-to-proc/
- https://likegeeks.com/secure-linux-server-hardening-best-practices/#Hardening-proc-Directory
- https://www.cyberciti.biz/faq/linux-hide-processes-from-other-users/

#### 操作步骤

1. 备份 `/etc/fstab`：

    ``` bash
    sudo cp --archive /etc/fstab /etc/fstab-COPY-$(date +"%Y%m%d%H%M%S")
    ```

1. 将下面一行添加到 `/etc/fstab`，使 `/proc` 以 `hidepid=2` 挂载：

    ```
    proc     /proc     proc     defaults,hidepid=2     0     0
    ```
    
    [给想省事的人](#editing-configuration-files---for-the-lazy)：
    
    ``` bash
    echo -e "\nproc     /proc     proc     defaults,hidepid=2     0     0         # added by $(whoami) on $(date +"%Y-%m-%d @ %H:%M:%S")" | sudo tee -a /etc/fstab
    ```

1. 重启系统：

    ``` bash
    sudo reboot now
    ```
    
    **注意**：也可以重新挂载 `/proc` 而无需重启，所用命令为 `sudo mount -o remount,hidepid=2 /proc`。

([目录](#table-of-contents))

<a id="force-accounts-to-use-secure-passwords"></a>
### 强制账户使用安全密码

#### 原因

默认情况下，账户可以使用任意密码，包括糟糕的密码。[pwquality](https://linux.die.net/man/5/pwquality.conf)/[pam_pwquality](https://linux.die.net/man/8/pam_pwquality) 提供了“一种配置系统密码默认质量要求的方法”，并“依据系统词典和一组用于识别不良选择的规则检查密码强度”，从而弥补这一安全缺口。

#### 工作原理

在 Linux 上，PAM 负责身份验证。PAM 有四类任务，可在 https://en.wikipedia.org/wiki/Linux_PAM 了解相关信息。本节讨论其中的密码任务。

需要设置或更改账户密码时，PAM 的密码任务会处理该请求。在本节中，我们将让 PAM 的密码任务把用户请求设置的新密码交给 libpam-pwquality，以确保密码符合要求。符合要求时，密码会被采用并设置；不符合要求时，则会返回错误并通知用户。

#### 目标

- 强制使用高强度密码

#### 操作步骤

1. 安装 libpam-pwquality。

    在基于 Debian 的系统上：

    ``` bash
    sudo apt install libpam-pwquality
    ```

1. 备份 PAM 的密码配置文件 `/etc/pam.d/common-password`：

    ``` bash
    sudo cp --archive /etc/pam.d/common-password /etc/pam.d/common-password-COPY-$(date +"%Y%m%d%H%M%S")
    ```

1. 编辑 `/etc/pam.d/common-password`，让 PAM 使用 libpam-pwquality 强制采用高强度密码。将以下述内容开头的行：

    ```
    password        requisite                       pam_pwquality.so
    ```

    更改为：

    ```
    password        requisite                       pam_pwquality.so retry=3 minlen=10 difok=3 ucredit=-1 lcredit=-1 dcredit=-1 ocredit=-1 maxrepeat=3 gecoscheck=1
    ```

   上述选项的含义如下：

     - `retry=3` = 返回错误前提示用户 3 次。
     - `minlen=10` = 密码最小长度，同时计入下列各项带来的加分（或减分）：
       - `dcredit=-1` = 必须至少包含**一个数字**
       - `ucredit=-1` = 必须至少包含**一个大写字母**
       - `lcredit=-1` = 必须至少包含**一个小写字母**
       - `ocredit=-1` = 必须至少包含**一个非字母数字字符**
     - `difok=3` = 新密码中至少要有 3 个字符未出现在旧密码中
     - `maxrepeat=3` = 最多允许 3 个重复字符
     - `gecoscheck=1` = 不允许密码中包含账户名称


    [给想省事的人](#editing-configuration-files---for-the-lazy)：
    
    ``` bash
    sudo sed -i -r -e "s/^(password\s+requisite\s+pam_pwquality.so)(.*)$/# \1\2         # commented by $(whoami) on $(date +"%Y-%m-%d @ %H:%M:%S")\n\1 retry=3 minlen=10 difok=3 ucredit=-1 lcredit=-1 dcredit=-1 ocredit=-1 maxrepeat=3 gecoscheck=1         # added by $(whoami) on $(date +"%Y-%m-%d @ %H:%M:%S")/" /etc/pam.d/common-password
    ```

([目录](#table-of-contents))

<a id="automatic-security-updates-and-alerts"></a>
### 自动安全更新与提醒

#### 原因

让服务器及时安装最新的**关键安全补丁和更新**非常重要。否则，你将面临已知安全漏洞带来的风险，恶意行为者可能利用这些漏洞未经授权访问服务器。

除非你打算每天检查服务器，否则需要一种自动更新系统和/或通过电子邮件获知可用更新的方法。

你不会希望自动安装所有更新，因为每次更新都有可能破坏某些功能。安装关键更新很重要，但其他更新可以等到你有时间时再手动处理。

#### 不采用的理由

自动更新和无人值守更新可能会破坏系统，而你当时可能不在服务器旁，无法进行修复。如果更新破坏了 SSH 访问，问题尤其严重。

#### 注意事项

- 每个发行版管理软件包和更新的方式都不同。目前我只提供基于 Debian 的系统的操作步骤。
- 要让此功能正常工作，服务器需要具备发送电子邮件的方式

#### 目标

- 自动、无人值守地安装关键安全补丁
- 自动通过电子邮件通知其余待处理更新

#### 基于 Debian 的系统

##### 工作原理

在基于 Debian 的系统上，可以使用：

- unattended-upgrades 自动执行你需要的系统更新（例如关键安全更新）
- apt-listchanges 在安装/升级软件包前获取其变更详情
- apticron 通过电子邮件通知待处理的软件包更新

我们将使用 unattended-upgrades 应用**关键安全补丁**。还可以应用 stable 更新，因为这些更新已经过 Debian 社区的充分测试。

##### 参考资料

- https://wiki.debian.org/UnattendedUpgrades
- https://debian-handbook.info/browse/stable/sect.regular-upgrades.html
- https://blog.sleeplessbeastie.eu/2015/01/02/how-to-perform-unattended-upgrades/
- https://www.vultr.com/docs/how-to-set-up-unattended-upgrades-on-debian-9-stretch
- https://github.com/mvo5/unattended-upgrades
- https://wiki.debian.org/UnattendedUpgrades#apt-listchanges
- https://www.cyberciti.biz/faq/apt-get-apticron-send-email-upgrades-available/
- https://www.unixmen.com/how-to-get-email-notifications-for-new-updates-on-debianubuntu/
- `/etc/apt/apt.conf.d/50unattended-upgrades`

##### 操作步骤

1. 安装 unattended-upgrades、apt-listchanges 和 apticron：

    ``` bash
    sudo apt install unattended-upgrades apt-listchanges apticron
    ```

1. 现在需要配置 unattended-upgrades，使其自动应用更新。通常的做法是编辑软件包创建的 `/etc/apt/apt.conf.d/20auto-upgrades` 和 `/etc/apt/apt.conf.d/50unattended-upgrades` 文件。不过，这些文件可能会被未来的更新覆盖，因此我们改为创建一个新文件。创建 `/etc/apt/apt.conf.d/51myunattended-upgrades` 并添加以下内容：

    ```
    // Enable the update/upgrade script (0=disable)
    APT::Periodic::Enable "1";

    // Do "apt-get update" automatically every n-days (0=disable)
    APT::Periodic::Update-Package-Lists "1";

    // Do "apt-get upgrade --download-only" every n-days (0=disable)
    APT::Periodic::Download-Upgradeable-Packages "1";

    // Do "apt-get autoclean" every n-days (0=disable)
    APT::Periodic::AutocleanInterval "7";

    // Send report mail to root
    //     0:  no report             (or null string)
    //     1:  progress report       (actually any string)
    //     2:  + command outputs     (remove -qq, remove 2>/dev/null, add -d)
    //     3:  + trace on    APT::Periodic::Verbose "2";
    APT::Periodic::Unattended-Upgrade "1";

    // Automatically upgrade packages from these
    Unattended-Upgrade::Origins-Pattern {
          "origin=Debian,codename=${distro_codename},label=Debian";
          "origin=Debian,codename=${distro_codename}-updates";
          "origin=Debian,codename=${distro_codename},label=Debian-Security";
          "origin=Debian,codename=${distro_codename}-security,label=Debian-Security";
    };

    // You can specify your own packages to NOT automatically upgrade here
    Unattended-Upgrade::Package-Blacklist {
    };

    // Run dpkg --force-confold --configure -a if a unclean dpkg state is detected to true to ensure that updates get installed even when the system got interrupted during a previous run
    Unattended-Upgrade::AutoFixInterruptedDpkg "true";

    //Perform the upgrade when the machine is running because we wont be shutting our server down often
    Unattended-Upgrade::InstallOnShutdown "false";

    // Send an email to this address with information about the packages upgraded.
    Unattended-Upgrade::Mail "root";

    // Always send an e-mail
    Unattended-Upgrade::MailOnlyOnError "false";

    // Remove all unused dependencies after the upgrade has finished
    Unattended-Upgrade::Remove-Unused-Dependencies "true";

    // Remove any new unused dependencies after the upgrade has finished
    Unattended-Upgrade::Remove-New-Unused-Dependencies "true";

    // Automatically reboot WITHOUT CONFIRMATION if the file /var/run/reboot-required is found after the upgrade.
    Unattended-Upgrade::Automatic-Reboot "true";

    // Automatically reboot even if users are logged in.
    Unattended-Upgrade::Automatic-Reboot-WithUsers "true";
    ```

    **注意事项**：
    - 请查看 `/usr/lib/apt/apt.systemd.daily`，了解 `APT::Periodic` 选项的详细信息
    - 有关 `Unattended-Upgrade` 选项的详细信息，请查看 https://github.com/mvo5/unattended-upgrades

1. 对 unattended-upgrades 执行一次试运行，确保配置文件没有问题：

    ``` bash
    sudo unattended-upgrade -d --dry-run
    ```

    如果一切正常，可以让它按计划运行，或使用 `unattended-upgrade -d` 强制运行一次。

1. 按照你的偏好配置 apt-listchanges：

    ``` bash
    sudo dpkg-reconfigure apt-listchanges
    ```

1. apticron 的默认设置已经足够好；如果想要修改，可以在 `/etc/apticron/apticron.conf` 中查看这些设置。例如，我的配置如下：

    > ```
    > EMAIL="root"
    > NOTIFY_NO_UPDATES="1"
    > ```

([目录](#table-of-contents))

<a id="more-secure-random-entropy-pool-wip"></a>
### 更安全的随机熵池（WIP）

#### 原因

WIP

#### 工作原理

WIP

#### 目标

WIP

#### 参考资料

- 感谢 [branneman](https://github.com/branneman) 在[问题 #33](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/issues/33)中提交这个想法。
- https://hackaday.com/2017/11/02/what-is-entropy-and-how-do-i-get-more-of-it/
- https://www.2uo.de/myths-about-urandom
- https://www.gnu.org/software/hurd/user/tlecarrour/rng-tools.html
- https://wiki.archlinux.org/index.php/Rng-tools
- https://www.howtoforge.com/helping-the-random-number-generator-to-gain-enough-entropy-with-rng-tools-debian-lenny
- https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/security_guide/sect-security_guide-encryption-using_the_random_number_generator

#### 操作步骤

1. 安装 rng-tools。
   
    在基于 Debian 的系统上：

    ``` bash
    sudo apt-get install rng-tools
    ```

1. 现在需要设置用于生成随机数的硬件设备。将以下内容添加到 `/etc/default/rng-tools`：

    ```
    HRNGDEVICE=/dev/urandom
    ```
    
    [给想省事的人](#editing-configuration-files---for-the-lazy)：
    
    ``` bash
    echo "HRNGDEVICE=/dev/urandom" | sudo tee -a /etc/default/rng-tools
    ```

1. 重启服务：

    ``` bash
    sudo systemctl stop rng-tools.service
    sudo systemctl start rng-tools.service
    ```

1. 测试随机性：
    - https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/security_guide/sect-security_guide-encryption-using_the_random_number_generator
    - https://wiki.archlinux.org/index.php/Rng-tools

([目录](#table-of-contents))

<a id="add-panicsecondaryfake-password-login-security-system"></a>
### 添加胁迫/备用/伪密码登录安全系统

#### 原因

这是一个不错的工具，可以增加额外的密码安全保护，用于抵御当面勒索、抢劫、袭击等物理攻击方式。

#### 工作原理

pamduress 会为用户 X 添加一个备用密码（胁迫密码）。当输入的密码与该胁迫密码匹配时，pamduress 会启动一个脚本；用户使用这个胁迫密码登录时，该脚本会执行你预先指定的操作。

实际且现实的示例：
“某个窃贼闯入一户人家，偷走了服务器（其中保存着重要的业务备份、个人生活回忆等等）。系统没有任何磁盘/启动加密。窃贼在自己的‘安全区域’启动服务器，并开始暴力破解。他通过 SSH 成功破解了拥有 sudo 权限的 'admin' 用户的本地密码；没错，那只是一个伪密码，而不是高强度的主密码。他使用这个已破解的伪密码/胁迫密码，以拥有 sudo 权限的 'admin' 用户发起 SSH 会话或物理会话。不到两分钟，他便开始觉得服务器异常繁忙，直到系统冻结……‘搞什么！？重启后继续窃取信息吧……’……抱歉，所有数据和系统都已被销毁。”
    结论：窃贼破解的是伪密码/胁迫密码/备用密码；与该密码关联的脚本会删除所有文件、配置、系统和启动内容，随后开始占满 RAM 和 CPU，迫使窃贼重启系统。  

#### 目标

防止恶意人员在通过强迫手段（袭击、枪支威胁、勒索等）获得密码后访问服务器信息。当然，这在其他情况下也很有帮助。

#### 参考资料

- 感谢 [nuvious](https://github.com/nuvious/pam-duress) 提供此工具
- 感谢 [hellresistor](https://gist.github.com/hellresistor/a4c542415a2d437e21afc235260d2366) 提供这个省事工具脚本

#### 操作步骤

1. 运行以下脚本（hellresistor 的省事工具脚本）。

 ```` bash
#!/bin/bash
myownscript(){
#######################################################
## ***** EDIT THIS SCRIPT TO YOUR PROPOSES *****#

cat > "$ScriptFile" <<-EOF
#!/bin/bash
sudo rm -rf /home
#### FINISHED OWN SCRIPT ####
EOF
#######################################################
}
echo "Lets Config a PANIC PASSWORD ;)" && sleep 1
read -r -p "Want you REALLY configure A PANIC PASSWORD?? Write [ OK ] : " PAMDUR
if [[ "$PAMDUR" = "OK" ]]; then
 echo "Lets Config a PANIC USER, PASSWORD and SCRIPT ;)" && sleep 1
 while [ -z "$PANICUSR" ]
 do
  read -r -p "WRITE a Panic User to your pam-duress user [ root ]: " PANICUSR
  PANICUSR=${PANICUSR:=root}
 done
 if [ -z "$ScriptLoc" ]; then
  read -r -p "SET Script Directory with FULL PATH [ /root/.duress ]: " ScriptLoc
  ScriptLoc=${ScriptLoc:=/root/.duress}
  ScriptFile="$ScriptLoc/PanicScript.sh"
 fi
else
 echo "NOT Use PAM DURESS aKa Panic Password!!! Bye"
 exit 1
fi

sudo apt install -y git build-essential libpam0g-dev libssl-dev

cd "$HOME" || exit 1
git clone https://github.com/nuvious/pam-duress.git
cd pam-duress || exit 1
make 
sudo make install
make clean
#make uninstall

mkdir -p $ScriptLoc
sudo mkdir -p /etc/duress.d
myownscript
duress_sign $ScriptFile
chmod -R 500 $ScriptLoc
chmod 400 $ScriptLoc/*.sha256
chown -R $PANICUSR $ScriptLoc

sudo cp --preserve /etc/pam.d/common-auth /etc/pam.d/common-auth.bck

echo "
auth   	[success=2 default=ignore]	     pam_unix.so nullok_secure
auth    [success=1 default=ignore]      pam_duress.so
auth	   requisite	                    		pam_deny.so
auth	   required	                     		pam_permit.so
" | sudo tee /etc/pam.d/common-auth

read -r -p "Press <Enter> Key to Finish PAM DURESS Script!"
exit 0
 ````

([目录](#table-of-contents))
<a id="the-network"></a>
## 网络

<a id="firewall-with-ufw-uncomplicated-firewall"></a>
### 使用 UFW（Uncomplicated Firewall）配置防火墙

<a id="why-14"></a>
#### 原因

你可以说我多疑，也不必赞同我的观点，但我希望拒绝所有进出服务器的流量，唯有我明确允许的流量除外。为什么我的服务器会发送我不知情的出站流量？如果我不知道外部流量来自谁或什么对象，它又为什么要尝试访问我的服务器？就良好的安全性而言，我的观点是默认拒绝，仅按例外情况允许。

当然，如果你不同意，完全没关系；你可以根据自己的需求配置 UFW。

无论如何，确保只有我们明确允许的流量能够通过，正是防火墙的职责。

<a id="how-it-works-9"></a>
#### 工作原理

Linux 内核提供了监控和控制网络流量的能力。这些能力通过防火墙实用工具提供给最终用户。在 Linux 上，最常见的防火墙是 [iptables](https://en.wikipedia.org/wiki/Iptables)。然而，iptables 相当复杂且令人困惑（依我之见）。这正是 UFW 的用武之地。可以把 UFW 看作 iptables 的前端。它简化了管理 iptables 规则的过程，而这些规则会告诉 Linux 内核如何处理网络流量。

**UFW** 允许你配置具有以下作用的规则：

- **允许**或**拒绝**
- **入站**或**出站**流量
- 流量**发往**或**来自**端口

你可以通过明确指定端口来创建规则，也可以使用指定了端口的应用程序配置来创建规则。

<a id="goals-14"></a>
#### 目标

 - 除明确允许的流量外，阻止所有入站和出站网络流量

<a id="notes-6"></a>
#### 注意事项

- 安装其他程序时，需要启用必要的端口/应用程序。

<a id="references-12"></a>
#### 参考资料

- https://launchpad.net/ufw

<a id="steps-14"></a>
#### 操作步骤

1. 安装 ufw。

    在基于 Debian 的系统上：

    ``` bash
    sudo apt install ufw
    ```

1. 拒绝所有出站流量：

    ``` bash
    sudo ufw default deny outgoing comment 'deny all outgoing traffic'
    ```

    > ```
    > Default outgoing policy changed to 'deny'
    > (be sure to update your rules accordingly)
    > ```

    如果你不像我这么多疑，不想拒绝所有出站流量，也可以改为允许所有出站流量：

    ``` bash
    sudo ufw default allow outgoing comment 'allow all outgoing traffic'
    ```

1. 拒绝所有入站流量：

    ``` bash
    sudo ufw default deny incoming comment 'deny all incoming traffic'
    ```

1. 显然，我们希望允许 SSH 连接进入。使用 limit 而不是 allow 时，如果某个 IP 地址在 30 秒的时间窗口内尝试发起 6 次或更多连接，系统会自动拒绝来自该 IP 地址的连接：

    ``` bash
    SSH_PORT="$(sudo sshd -T | awk '/^port / { print $2; exit }')"
    sudo ufw limit in "${SSH_PORT}/tcp" comment 'allow SSH connections in'
    ```

    > ```
    > Rules updated
    > Rules updated (v6)
    > ```

1. 根据需要允许其他流量。一些常见使用场景如下：

    ``` bash
    # allow traffic out to port 53 -- DNS
    sudo ufw allow out 53 comment 'allow DNS calls out'
	
	# allow traffic out to port 123 -- NTP
    sudo ufw allow out 123 comment 'allow NTP out'

    # allow traffic out for HTTP, HTTPS, or FTP
    # apt might needs these depending on which sources you're using
    sudo ufw allow out http comment 'allow HTTP traffic out'
    sudo ufw allow out https comment 'allow HTTPS traffic out'
    sudo ufw allow out ftp comment 'allow FTP traffic out'

    # allow whois
    sudo ufw allow out whois comment 'allow whois'
    
    # allow mails for status notifications -- choose port according to your provider
    sudo ufw allow out 25 comment 'allow SMTP out'
    sudo ufw allow out 587 comment 'allow SMTP out'

    # allow traffic out to port 68 -- the DHCP client
    # you only need this if you're using DHCP
    sudo ufw allow out 67 comment 'allow the DHCP client to update'
    sudo ufw allow out 68 comment 'allow the DHCP client to update'
    ```
    
    **注意**：安装软件包以及执行许多其他操作时，需要允许 HTTP/HTTPS。

1. 启动 ufw：

    ``` bash
    sudo ufw enable
    ```

    > ```
    > Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
    > Firewall is active and enabled on system startup
    > ```

1. 如果要查看状态：

    ``` bash
    sudo ufw status
    ```

    > ```
    > Status: active
    > 
    > To                         Action      From
    > --                         ------      ----
    > 22/tcp                     LIMIT       Anywhere                   # allow SSH connections in
    > 22/tcp (v6)                LIMIT       Anywhere (v6)              # allow SSH connections in
    > 
    > 53                         ALLOW OUT   Anywhere                   # allow DNS calls out
    > 123                        ALLOW OUT   Anywhere                   # allow NTP out
    > 80/tcp                     ALLOW OUT   Anywhere                   # allow HTTP traffic out
    > 443/tcp                    ALLOW OUT   Anywhere                   # allow HTTPS traffic out
    > 21/tcp                     ALLOW OUT   Anywhere                   # allow FTP traffic out
    > Mail submission            ALLOW OUT   Anywhere                   # allow mail out
    > 43/tcp                     ALLOW OUT   Anywhere                   # allow whois
    > 53 (v6)                    ALLOW OUT   Anywhere (v6)              # allow DNS calls out
    > 123 (v6)                   ALLOW OUT   Anywhere (v6)              # allow NTP out
    > 80/tcp (v6)                ALLOW OUT   Anywhere (v6)              # allow HTTP traffic out
    > 443/tcp (v6)               ALLOW OUT   Anywhere (v6)              # allow HTTPS traffic out
    > 21/tcp (v6)                ALLOW OUT   Anywhere (v6)              # allow FTP traffic out
    > Mail submission (v6)       ALLOW OUT   Anywhere (v6)              # allow mail out
    > 43/tcp (v6)                ALLOW OUT   Anywhere (v6)              # allow whois
    > ```

    或

    ``` bash
    sudo ufw status verbose
    ```

    > ```
    > Status: active
    > Logging: on (low)
    > Default: deny (incoming), deny (outgoing), disabled (routed)
    > New profiles: skip
    > 
    > To                         Action      From
    > --                         ------      ----
    > 22/tcp                     LIMIT IN    Anywhere                   # allow SSH connections in
    > 22/tcp (v6)                LIMIT IN    Anywhere (v6)              # allow SSH connections in
    > 
    > 53                         ALLOW OUT   Anywhere                   # allow DNS calls out
    > 123                        ALLOW OUT   Anywhere                   # allow NTP out
    > 80/tcp                     ALLOW OUT   Anywhere                   # allow HTTP traffic out
    > 443/tcp                    ALLOW OUT   Anywhere                   # allow HTTPS traffic out
    > 21/tcp                     ALLOW OUT   Anywhere                   # allow FTP traffic out
    > 587/tcp (Mail submission)  ALLOW OUT   Anywhere                   # allow mail out
    > 43/tcp                     ALLOW OUT   Anywhere                   # allow whois
    > 53 (v6)                    ALLOW OUT   Anywhere (v6)              # allow DNS calls out
    > 123 (v6)                   ALLOW OUT   Anywhere (v6)              # allow NTP out
    > 80/tcp (v6)                ALLOW OUT   Anywhere (v6)              # allow HTTP traffic out
    > 443/tcp (v6)               ALLOW OUT   Anywhere (v6)              # allow HTTPS traffic out
    > 21/tcp (v6)                ALLOW OUT   Anywhere (v6)              # allow FTP traffic out
    > 587/tcp (Mail submission (v6)) ALLOW OUT   Anywhere (v6)              # allow mail out
    > 43/tcp (v6)                ALLOW OUT   Anywhere (v6)              # allow whois
    > ```

7. 如果需要删除规则
    
    ``` bash
    sudo ufw status numbered
    [...]
    sudo ufw delete 3 #line number of the rule you want to delete
    ```

<a id="default-applications"></a>
#### 默认应用程序

ufw 随附一些默认应用程序。可以通过以下命令查看：

``` bash
sudo ufw app list
```

> ```
> Available applications:
>   AIM
>   Bonjour
>   CIFS
>   DNS
>   Deluge
>   IMAP
>   IMAPS
>   IPP
>   KTorrent
>   Kerberos Admin
>   Kerberos Full
>   Kerberos KDC
>   Kerberos Password
>   LDAP
>   LDAPS
>   LPD
>   MSN
>   MSN SSL
>   Mail submission
>   NFS
>   OpenSSH
>   POP3
>   POP3S
>   PeopleNearby
>   SMTP
>   SSH
>   Socks
>   Telnet
>   Transmission
>   Transparent Proxy
>   VNC
>   WWW
>   WWW Cache
>   WWW Full
>   WWW Secure
>   XMPP
>   Yahoo
>   qBittorrent
>   svnserve
> ```

要获取应用程序的详细信息，例如其中包含哪些端口，请输入：

``` bash
sudo ufw app info [app name]
```

> ``` bash
> sudo ufw app info DNS
> ```
> 
> ```
> Profile: DNS
> Title: Internet Domain Name Server
> Description: Internet Domain Name Server
> 
> Port:
>   53
> ```

<a id="custom-application"></a>
#### 自定义应用程序

如果不想通过明确提供端口号来创建规则，可以创建自己的应用程序配置。为此，请在 `/etc/ufw/applications.d` 中创建一个文件。

例如，下面是可用于 [Plex](https://support.plex.tv/articles/201543147-what-network-ports-do-i-need-to-allow-through-my-firewall/) 的配置：

``` bash
cat /etc/ufw/applications.d/plexmediaserver
```

> ```
> [PlexMediaServer]
> title=Plex Media Server
> description=This opens up PlexMediaServer for http (32400), upnp, and autodiscovery.
> ports=32469/tcp|32413/udp|1900/udp|32400/tcp|32412/udp|32410/udp|32414/udp|32400/udp
> ```

然后，就可以像启用其他任何应用程序一样启用它：

```bash
sudo ufw allow plexmediaserver
```

([目录](#table-of-contents))

<a id="iptables-intrusion-detection-and-prevention-with-psad"></a>
### 使用 PSAD 和 iptables 进行入侵检测与防御

<a id="why-15"></a>
#### 原因

即使有防火墙守住各个入口，攻击者仍可能尝试对任何受保护的入口进行暴力破解。我们希望监控所有网络活动，以检测并阻止潜在的入侵尝试，例如反复尝试进入系统。

<a id="how-it-works-10"></a>
#### 工作原理

我无法比 [FINESEC](https://serverfault.com/users/143961/finesec) 这位来自 https://serverfault.com/ 的用户在 https://serverfault.com/a/447604/289829 中解释得更好。

> Fail2BAN 会扫描 apache、ssh 或 ftp 等各种应用程序的日志文件，并自动封禁表现出恶意迹象（例如自动登录尝试）的 IP。另一方面，PSAD 会扫描 iptables 和 ip6tables 日志消息（通常位于 /var/log/messages），以检测扫描及 DDoS 或操作系统指纹识别尝试等其他类型的可疑流量，并可选择将其阻止。可以同时使用这两个程序，因为它们在不同层面运行。

此外，由于我们已经在使用 [UFW](#firewall-with-ufw-uncomplicated-firewall)，因此将按照 [netson](https://gist.github.com/netson) 在 https://gist.github.com/netson/c45b2dc4e835761fbccc 提供的出色说明，让 PSAD 与 UFW 配合工作。

<a id="references-13"></a>
#### 参考资料

- http://www.cipherdyne.org/psad/
- http://www.cipherdyne.org/psad/docs/config.html
- https://www.thefanclub.co.za/how-to/how-install-psad-intrusion-detection-ubuntu-1204-lts-server
- https://serverfault.com/a/447604/289829
- https://serverfault.com/a/770424/289829
- https://gist.github.com/netson/c45b2dc4e835761fbccc
- 感谢 [moltenbit](https://github.com/moltenbit) 发现 `psadwatchd` 的问题（[#61](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/issues/61)）。

<a id="steps-15"></a>
#### 操作步骤

1. 安装 psad。

    在基于 Debian 的系统上：

    ``` bash
    sudo apt install psad
    ```

1. 备份 psad 的配置文件 `/etc/psad/psad.conf`：

    ``` bash
    sudo cp --archive /etc/psad/psad.conf /etc/psad/psad.conf-COPY-$(date +"%Y%m%d%H%M%S")
    ```

1. 检查并更新 `/etc/psad/psad.conf` 中的配置选项。请特别注意以下选项：

   |设置|设为
   |--|--|
    |[`EMAIL_ADDRESSES`](http://www.cipherdyne.org/psad/docs/config.html#EMAIL_ADDRESSES)|你的电子邮件地址（可填写一个或多个）|
   |`HOSTNAME`|服务器的主机名|
   |`EXPECT_TCP_OPTIONS`|`EXPECT_TCP_OPTIONS Y;`|
   |`ENABLE_PSADWATCHD`|`ENABLE_PSADWATCHD Y;`|
   |[`ENABLE_AUTO_IDS`](http://www.cipherdyne.org/psad/docs/config.html#ENABLE_AUTO_IDS)|`ENABLE_AUTO_IDS Y;`|
   |`ENABLE_AUTO_IDS_EMAILS`|`ENABLE_AUTO_IDS_EMAILS Y;`|

   有关配置文件的更多详细信息，请查阅位于 http://www.cipherdyne.org/psad/docs/config.html 的 psad 文档。

1. <a name="psad_step4"></a>现在需要对 ufw 做一些更改，使其能够与 psad 配合工作：让 ufw 记录所有流量，以便 psad 对其进行分析。为此，请编辑**两个文件**，并将以下行添加到**文件末尾、COMMIT 行之前**。

    创建备份：

    ``` bash
    sudo cp --archive /etc/ufw/before.rules /etc/ufw/before.rules-COPY-$(date +"%Y%m%d%H%M%S")
    sudo cp --archive /etc/ufw/before6.rules /etc/ufw/before6.rules-COPY-$(date +"%Y%m%d%H%M%S")
    ```

    编辑以下文件：

    - `/etc/ufw/before.rules`
    - `/etc/ufw/before6.rules`

    将以下内容添加到**文件末尾、COMMIT 行之前**：

    ```
    # log all traffic so psad can analyze
    -A INPUT -j LOG --log-tcp-options --log-prefix "[IPTABLES] "
    -A FORWARD -j LOG --log-tcp-options --log-prefix "[IPTABLES] "
    ```

    **注意**：我们正在为所有 iptables 日志添加日志前缀。之后需要用它来[将 iptables 日志分离到单独的文件中](#separate-iptables-log-file)。

    例如：

    > ```
    > ...
    > 
    > # log all traffic so psad can analyze
    > -A INPUT -j LOG --log-tcp-options --log-prefix "[IPTABLES] "
    > -A FORWARD -j LOG --log-tcp-options --log-prefix "[IPTABLES] "
    > 
    > # don't delete the 'COMMIT' line or these rules won't be processed
    > COMMIT
    > ```

1. 现在需要重新加载/重新启动 ufw 和 psad，使更改生效：

    ``` bash
    sudo ufw reload

    sudo psad -R
    sudo psad --sig-update
    sudo psad -H
    ```

1. 分析 iptables 规则是否存在错误：

    ``` bash
    sudo psad --fw-analyze
    ```

    > ```
    > [+] Parsing INPUT chain rules.
    > [+] Parsing INPUT chain rules.
    > [+] Firewall config looks good.
    > [+] Completed check of firewall ruleset.
    > [+] Results in /var/log/psad/fw_check
    > [+] Exiting.
    > ```

    **注意**：如果存在任何问题，你会收到一封包含错误信息的电子邮件。

1. 检查 psad 的状态：

    ``` bash
    sudo psad --Status
    ```

    > ```
    > [-] psad: pid file /var/run/psad/psadwatchd.pid does not exist for psadwatchd on vm
    > [+] psad_fw_read (pid: 3444)  %CPU: 0.0  %MEM: 2.2
    >     Running since: Sat Feb 16 01:03:09 2019
    > 
    > [+] psad (pid: 3435)  %CPU: 0.2  %MEM: 2.7
    >     Running since: Sat Feb 16 01:03:09 2019
    >     Command line arguments: [none specified]
    >     Alert email address(es): root@localhost
    > 
    > [+] Version: psad v2.4.3
    > 
    > [+] Top 50 signature matches:
    >         [NONE]
    > 
    > [+] Top 25 attackers:
    >         [NONE]
    > 
    > [+] Top 20 scanned ports:
    >         [NONE]
    > 
    > [+] iptables log prefix counters:
    >         [NONE]
    > 
    >     Total protocol packet counters:
    > 
    > [+] IP Status Detail:
    >         [NONE]
    > 
    >     Total scan sources: 0
    >     Total scan destinations: 0
    > 
    > [+] These results are available in: /var/log/psad/status.out
    > ```

([目录](#table-of-contents))

<a id="application-intrusion-detection-and-prevention-with-fail2ban"></a>
### 使用 Fail2Ban 进行应用程序入侵检测与防御

<a id="why-16"></a>
#### 原因

UFW 会告诉服务器应封住哪些入口，使任何人都无法看到它们，以及应允许授权用户通过哪些入口。PSAD 会监控网络活动，以检测和防止潜在入侵——即反复尝试进入系统。

但是，对于服务器上运行且防火墙已配置为允许访问的应用程序/服务（例如 SSH 和 Apache），该怎么办？即使允许访问，也不表示所有访问尝试都正当且无害。如果有人试图暴力破解服务器上运行的 Web 应用程序，该怎么办？这正是 Fail2ban 的用武之地。

<a id="how-it-works-11"></a>
#### 工作原理

Fail2ban 会监控应用程序（例如 SSH 和 Apache）的日志，以检测并防止潜在入侵。它会监控网络流量/日志，并通过阻止可疑活动（例如短时间内连续多次连接失败）来防止入侵。

<a id="goals-15"></a>
#### 目标

- 监控网络中的可疑活动，并自动封禁实施可疑活动的 IP 地址

<a id="notes-7"></a>
#### 注意事项

- 目前，这台服务器上唯一运行的服务是 SSH，因此需要让 Fail2ban 监控 SSH，并在必要时执行封禁。
- 安装其他程序时，需要创建/配置适当的 jail 并启用它们。

<a id="references-14"></a>
#### 参考资料

- https://www.fail2ban.org/
- https://blog.vigilcode.com/2011/05/ufw-with-fail2ban-quick-secure-setup-part-ii/
- https://dodwell.us/security/ufw-fail2ban-portscan.html
- https://www.howtoforge.com/community/threads/fail2ban-and-ufw-on-debian.77261/

<a id="steps-16"></a>
#### 操作步骤

1. 安装 fail2ban。

    在基于 Debian 的系统上：

    ``` bash
    sudo apt install fail2ban
    ```

1. 我们不希望编辑 `/etc/fail2ban/fail2ban.conf` 或 `/etc/fail2ban/jail.conf`，因为未来的更新可能会覆盖这些文件，所以改为创建一个本地副本。创建文件 `/etc/fail2ban/jail.local`，将 `[LAN SEGMENT]` 和 `[your email]` 替换为适当的值后，把以下内容添加到该文件：

    ```
    [DEFAULT]
    # the IP address range we want to ignore
    ignoreip = 127.0.0.1/8 [LAN SEGMENT]

    # who to send e-mail to
    destemail = [your e-mail]

    # who is the email from
    sender = [your e-mail]

    # since we're using exim4 to send emails
    mta = mail

    # get email alerts
    action = %(action_mwl)s
    ```

    **注意**：服务器必须能够发送电子邮件，Fail2ban 才能通知你可疑活动以及它何时封禁了某个 IP。

1. 需要为 SSH 创建一个 jail，指示 fail2ban 查看 SSH 日志，并根据需要使用 ufw 封禁/解除封禁 IP。创建文件 `/etc/fail2ban/jail.d/ssh.local`，并向其中添加以下内容，以创建 SSH jail：

    ```
    [sshd]
    enabled = true
    banaction = ufw
    port = ssh
    filter = sshd
    logpath = %(sshd_log)s
    maxretry = 5
    ```

    [懒人版](#editing-configuration-files---for-the-lazy)：

    ``` bash
    cat << EOF | sudo tee /etc/fail2ban/jail.d/ssh.local
    [sshd]
    enabled = true
    banaction = ufw
    port = ssh
    filter = sshd
    logpath = %(sshd_log)s
    maxretry = 5
    EOF
    ```

1. 在上面的配置中，我们让 fail2ban 使用 ufw 作为 `banaction`。Fail2ban 随附一个用于 ufw 的动作配置文件，可在 `/etc/fail2ban/action.d/ufw.conf` 中查看。

1. 启用 fail2ban：

    ``` bash
    sudo fail2ban-client start
    sudo fail2ban-client reload
    sudo fail2ban-client add sshd # This may fail on some systems if the sshd jail was added by default
    ```

1. 检查状态：

    ``` bash
    sudo fail2ban-client status
    ```

    > ```
    > Status
    > |- Number of jail:      1
    > `- Jail list:   sshd
    > ```

    ``` bash
    sudo fail2ban-client status sshd
    ```

    > ```
    > Status for the jail: sshd
    > |- Filter
    > |  |- Currently failed: 0
    > |  |- Total failed:     0
    > |  `- File list:        /var/log/auth.log
    > `- Actions
    >    |- Currently banned: 0
    >    |- Total banned:     0
    >    `- Banned IP list:
    > ```

<a id="custom-jails"></a>
#### 自定义 jail

我还不曾需要创建自定义 jail。等到需要并弄清楚如何创建时，我会更新本指南。或者，如果你知道如何创建，请帮助[贡献内容](#contributing)。

<a id="unban-an-ip"></a>
#### 解除 IP 封禁

要解除 IP 封禁，请使用以下命令：

``` bash
fail2ban-client set [jail] unbanip [IP]
```

`[jail]` 是封禁了该 IP 的 jail 名称，`[IP]` 是要解除封禁的 IP 地址。例如，要从 SSH 的封禁列表中解除对 `192.168.1.100` 的封禁，请执行：

``` bash
fail2ban-client set sshd unbanip 192.168.1.100
```

([目录](#table-of-contents))

<a id="application-intrusion-detection-and-prevention-with-crowdsec"></a>
### 使用 CrowdSec 进行应用程序入侵检测与防御

<a id="why-17"></a>
#### 原因

UFW 会告诉服务器应封住哪些入口，使任何人都无法看到它们，以及应允许授权用户通过哪些入口。PSAD 会监控网络活动，以检测和防止潜在入侵——即反复尝试进入系统。

CrowdSec 与 Fail2Ban 类似，它会监控应用程序（例如 SSH 和 Apache）的日志，以检测并防止潜在入侵。不过，CrowdSec 还与一个社区相结合，该社区会把威胁情报共享回 CrowdSec，随后由 CrowdSec 向所有用户分发社区阻止列表。

<a id="how-it-works-12"></a>
#### 工作原理

CrowdSec 会监控应用程序（例如 SSH 和 Apache）的日志，以检测并防止潜在入侵。它会监控网络流量/日志，并通过阻止可疑活动（例如短时间内连续多次连接失败）来防止入侵。一旦检测到恶意 IP，该 IP 就会被添加到本地决策列表，并且其威胁信息会共享给 CrowdSec，用于更新恶意 IP 地址的社区阻止列表。一旦某个 IP 地址的恶意活动达到一定阈值，该恶意 IP 的信息就会自动分发给所有其他 CrowdSec 用户，以便这些用户主动封禁该 IP。

<a id="goals-16"></a>
#### 目标

- 监控网络中的可疑活动，并自动封禁实施可疑活动的 IP 地址

<a id="notes-8"></a>
#### 注意事项

- 目前，这台服务器上唯一运行的服务是 SSH，因此需要让 CrowdSec 监控 SSH，并在必要时执行封禁。
- 安装其他程序时，需要安装其他集合并配置相应的数据采集项。

<a id="references-15"></a>
#### 参考资料

- https://www.crowdsec.net/
- [了解 CrowdSec 如何管理社区阻止列表](https://www.crowdsec.net/our-data)
- [了解哪些威胁情报会与 CrowdSec 共享](https://docs.crowdsec.net/docs/next/central_api/intro#signal-meta-data)
- https://docs.crowdsec.net/

<a id="steps-17"></a>
#### 操作步骤

1. 安装 CrowdSec Security Engine。（IDS）

    在任何 Linux 发行版上（包括基于 Debian 的系统）
    
    安装 CrowdSec 软件仓库：
    ``` bash
    curl -s https://install.crowdsec.net | sudo sh
    ```

    安装 CrowdSec Security Engine：
    ``` bash
    sudo apt install crowdsec
    ```

> [!TIP]
> 如果你不喜欢 `curl | sh`，可以在[此处](https://docs.crowdsec.net/u/getting_started/installation/linux)找到其他安装方法。

默认情况下，CrowdSec 安装 Security Engine 时会自动发现已安装的应用程序，并为它们安装适当的解析器和场景。由于大多数 Linux 服务器开箱即运行 ssh，CrowdSec 会自动为你完成相关配置。

2. 安装 Remediation Component。（IPS）

    CrowdSec 本身是一个检测引擎。由于在大多数现代基础设施中可能存在上游防火墙或 WAF，CrowdSec 不会自行阻止 IP 地址。可以安装 Remediation Component，阻止 CrowdSec 检测到的 IP 地址。
    ```bash
    sudo apt install crowdsec-firewall-bouncer-iptables
    ```

> [!TIP]
> 如果安装的 UFW 未使用 `iptables` 作为后端，也可以安装 `crowdsec-firewall-bouncer-nftables`。安装的二进制文件没有区别，只有配置文件不同。

默认情况下，如果 Remediation Component 与 Security Engine 部署在同一主机上（并且 Security Engine 不在容器环境中），安装 Remediation Component 时会自动配置与 Security Engine 配合工作所需的设置。

3. 检查检测和补救功能是否按预期工作：

    CrowdSec 软件包随附一个 CLI 工具，可用于检查 Security Engine 和 Remediation Component 的状态。

    ```bash
    sudo cscli metrics
    ```

    ```bash
    Acquisition Metrics:
    ╭────────────────────────┬────────────┬──────────────┬────────────────┬────────────────────────┬───────────────────╮
    │ Source                 │ Lines read │ Lines parsed │ Lines unparsed │ Lines poured to bucket │ Lines whitelisted │
    ├────────────────────────┼────────────┼──────────────┼────────────────┼────────────────────────┼───────────────────┤
    │ file:/var/log/auth.log │ 5          │ 4            │ 1              │ 10                     │ -                 │
    │ file:/var/log/syslog   │ 30         │ -            │ 30             │ -                      │ -                 │
    ╰────────────────────────┴────────────┴──────────────┴────────────────┴────────────────────────┴───────────────────╯

    Local API Decisions:
    ╭────────────────────────────────────────────┬────────┬────────┬───────╮
    │ Reason                                     │ Origin │ Action │ Count │
    ├────────────────────────────────────────────┼────────┼────────┼───────┤
    │ crowdsecurity/http-backdoors-attempts      │ CAPI   │ ban    │ 73    │
    │ crowdsecurity/http-bad-user-agent          │ CAPI   │ ban    │ 4836  │
    │ crowdsecurity/http-path-traversal-probing  │ CAPI   │ ban    │ 87    │
    │ crowdsecurity/http-probing                 │ CAPI   │ ban    │ 2010  │
    │ crowdsecurity/thinkphp-cve-2018-20062      │ CAPI   │ ban    │ 88    │
    │ crowdsecurity/CVE-2019-18935               │ CAPI   │ ban    │ 7     │
    │ crowdsecurity/CVE-2023-49103               │ CAPI   │ ban    │ 5     │
    │ crowdsecurity/http-admin-interface-probing │ CAPI   │ ban    │ 91    │
    │ ltsich/http-w00tw00t                       │ CAPI   │ ban    │ 3     │
    │ crowdsecurity/apache_log4j2_cve-2021-44228 │ CAPI   │ ban    │ 18    │
    │ crowdsecurity/nginx-req-limit-exceeded     │ CAPI   │ ban    │ 280   │
    │ crowdsecurity/ssh-slow-bf                  │ CAPI   │ ban    │ 3412  │
    │ crowdsecurity/spring4shell_cve-2022-22965  │ CAPI   │ ban    │ 1     │
    │ crowdsecurity/ssh-cve-2024-6387            │ CAPI   │ ban    │ 24    │
    │ crowdsecurity/CVE-2023-22515               │ CAPI   │ ban    │ 2     │
    │ crowdsecurity/http-cve-2021-41773          │ CAPI   │ ban    │ 172   │
    │ crowdsecurity/netgear_rce                  │ CAPI   │ ban    │ 14    │
    │ crowdsecurity/ssh-bf                       │ CAPI   │ ban    │ 2000  │
    │ crowdsecurity/CVE-2022-35914               │ CAPI   │ ban    │ 1     │
    │ crowdsecurity/http-cve-2021-42013          │ CAPI   │ ban    │ 2     │
    │ crowdsecurity/jira_cve-2021-26086          │ CAPI   │ ban    │ 9     │
    │ crowdsecurity/http-sensitive-files         │ CAPI   │ ban    │ 166   │
    │ crowdsecurity/http-wordpress-scan          │ CAPI   │ ban    │ 272   │
    │ crowdsecurity/CVE-2022-26134               │ CAPI   │ ban    │ 5     │
    │ crowdsecurity/http-generic-bf              │ CAPI   │ ban    │ 7     │
    │ crowdsecurity/http-open-proxy              │ CAPI   │ ban    │ 948   │
    │ crowdsecurity/http-crawl-non_statics       │ CAPI   │ ban    │ 339   │
    │ crowdsecurity/http-cve-probing             │ CAPI   │ ban    │ 5     │
    │ crowdsecurity/CVE-2017-9841                │ CAPI   │ ban    │ 117   │
    │ crowdsecurity/CVE-2022-37042               │ CAPI   │ ban    │ 1     │
    │ crowdsecurity/fortinet-cve-2018-13379      │ CAPI   │ ban    │ 5     │
    ╰────────────────────────────────────────────┴────────┴────────┴───────╯

    Local API Metrics:
    ╭──────────────────────┬────────┬──────╮
    │ Route                │ Method │ Hits │
    ├──────────────────────┼────────┼──────┤
    │ /v1/alerts           │ GET    │ 2    │
    │ /v1/decisions/stream │ GET    │ 5    │
    │ /v1/usage-metrics    │ POST   │ 2    │
    │ /v1/watchers/login   │ POST   │ 4    │
    ╰──────────────────────┴────────┴──────╯

    Local API Bouncers Metrics:
    ╭────────────────────────────────┬──────────────────────┬────────┬──────╮
    │ Bouncer                        │ Route                │ Method │ Hits │
    ├────────────────────────────────┼──────────────────────┼────────┼──────┤
    │ cs-firewall-bouncer-1729025592 │ /v1/decisions/stream │ GET    │ 5    │
    ╰────────────────────────────────┴──────────────────────┴────────┴──────╯

    Local API Machines Metrics:
    ╭──────────────────────────────────────────────────┬────────────┬────────┬──────╮
    │ Machine                                          │ Route      │ Method │ Hits │
    ├──────────────────────────────────────────────────┼────────────┼────────┼──────┤
    │ <your_machine_id_will_be_here>                   │ /v1/alerts │ GET    │ 2    │
    ╰──────────────────────────────────────────────────┴────────────┴────────┴──────╯

    Parser Metrics:
    ╭─────────────────────────────────┬──────┬────────┬──────────╮
    │ Parsers                         │ Hits │ Parsed │ Unparsed │
    ├─────────────────────────────────┼──────┼────────┼──────────┤
    │ child-crowdsecurity/sshd-logs   │ 41   │ 4      │ 37       │
    │ child-crowdsecurity/syslog-logs │ 35   │ 35     │ -        │
    │ crowdsecurity/dateparse-enrich  │ 4    │ 4      │ -        │
    │ crowdsecurity/sshd-logs         │ 5    │ 4      │ 1        │
    │ crowdsecurity/syslog-logs       │ 35   │ 35     │ -        │
    ╰─────────────────────────────────┴──────┴────────┴──────────╯

    Scenario Metrics:
    ╭─────────────────────────────────────┬───────────────┬───────────┬──────────────┬────────┬─────────╮
    │ Scenario                            │ Current Count │ Overflows │ Instantiated │ Poured │ Expired │
    ├─────────────────────────────────────┼───────────────┼───────────┼──────────────┼────────┼─────────┤
    │ crowdsecurity/ssh-bf                │ 1             │ -         │ 1            │ 4      │ -       │
    │ crowdsecurity/ssh-bf_user-enum      │ 1             │ -         │ 1            │ 1      │ -       │
    │ crowdsecurity/ssh-slow-bf           │ 1             │ -         │ 1            │ 4      │ -       │
    │ crowdsecurity/ssh-slow-bf_user-enum │ 1             │ -         │ 1            │ 1      │ -       │
    ╰─────────────────────────────────────┴───────────────┴───────────┴──────────────┴────────┴─────────╯
    ```

上面的输出可能让人望而生畏，但它是检查 Security Engine 是否正在读取日志、Remediation Component 是否正在阻止 IP 地址的好方法。下面快速说明各个部分：

- **Acquisition Metrics**：本部分显示 Security Engine 正在读取和解析的日志。如果在 `Lines unparsed` 列中看到日志，则表示 Security Engine 无法解析这些日志。这可能是配置错误造成的，也可能是日志不符合预期格式。
- **Local API Decisions**：本部分显示 Security Engine 在数据库中保存的决策。如果在 `Count` 列中看到记录，则表示 Security Engine 已检测到恶意活动并阻止了相应 IP 地址。
    - 来源：表示决策来自哪里。在本例中，它来自 Central API（CAPI）。
- **Local API Metrics**：本部分显示 Local API 的访问次数。Security Engine 使用该 API 与 Remediation Component 通信。
- **Local API Bouncers Metrics**：本部分显示 Remediation Component 对 Local API 的访问次数。
- **Local API Machines Metrics**：本部分显示 Security Engine 对 Local API 的访问次数（如果在集中式配置中运行多个 Security Engine，可以在此看到多个 ID）。
- **Parser Metrics**：本部分显示 Security Engine 正在使用的解析器。如果在 `Unparsed` 列中看到日志，则表示 Security Engine 无法解析这些日志。这可能是配置错误造成的，也可能是日志不符合预期格式。
- **Scenario Metrics**：本部分显示 Security Engine 正在使用的场景。如果在 `Current Count` 列中看到记录，则表示 Security Engine 已检测到恶意活动，并正在跟踪相应 IP 地址。

<a id="unban-an-ip-1"></a>
#### 解除 IP 封禁

要解除 IP 封禁，请使用以下命令：

``` bash
cscli decisions delete --ip [IP]
```

`[IP]` 是要解除封禁的 IP 地址。例如，要从 SSH 的封禁列表中解除对 `192.168.1.100` 的封禁，请执行：

``` bash
cscli decisions delete --ip 192.168.1.100
```

<a id="the-auditing"></a>
## 审计

<a id="filefolder-integrity-monitoring-with-aide-wip"></a>
### 使用 AIDE 监控文件/文件夹完整性（WIP）

<a id="why-18"></a>
#### 原因

WIP

<a id="how-it-works-13"></a>
#### 工作原理

WIP

<a id="goals-17"></a>
#### 目标

WIP

<a id="references-16"></a>
#### 参考资料

- https://aide.github.io/
- https://www.hiroom2.com/2017/06/09/debian-8-file-integrity-check-with-aide/
- https://blog.rapid7.com/2017/06/30/how-to-install-and-configure-aide-on-ubuntu-linux/
- https://www.stephenrlang.com/2016/03/using-aide-for-file-integrity-monitoring-fim-on-ubuntu/
- https://www.howtoforge.com/how-to-configure-the-aide-advanced-intrusion-detection-environment-file-integrity-scanner-for-your-website
- https://www.tecmint.com/check-integrity-of-file-and-directory-using-aide-in-linux/
- https://www.cyberciti.biz/faq/debian-ubuntu-linux-software-integrity-checking-with-aide/
- https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/issues/83

<a id="steps-18"></a>
#### 操作步骤

1. 安装 AIDE。

    在基于 Debian 的系统上：
    
    ``` bash
    sudo apt install aide aide-common
    ```
    
1. 备份 AIDE 的默认设置文件：

    ``` bash
    sudo cp -p /etc/default/aide /etc/default/aide-COPY-$(date +"%Y%m%d%H%M%S")
    ```

1. 检查 `/etc/default/aide`，并根据自己的需求设置 AIDE 的默认值。如果希望 AIDE 每天运行并向你发送电子邮件，请务必将 `CRON_DAILY_RUN` 设为 `yes`。

1. 备份 AIDE 的配置文件：

    ``` bash
    sudo cp -pr /etc/aide /etc/aide-COPY-$(date +"%Y%m%d%H%M%S")
    ```

1. 在基于 Debian 的系统上：

    - AIDE 的配置文件位于 `/etc/aide/aide.conf.d/`。
    - 请查阅 AIDE 的文档并检查配置文件，按照自己的需求进行设置。
    - 如果需要添加新设置，例如监控新的文件夹，请将其添加到 `/etc/aide/aide.conf` 或 `/etc/aide/aide.conf.d/`。
    - 备份原始配置文件：`sudo cp -pr /etc/aide /etc/aide-COPY-$(date +"%Y%m%d%H%M%S")`。

1. 创建并安装新数据库。
   
    在基于 Debian 的系统上：

    ``` bash
    sudo aideinit
    ```
    
    > ```
    > Running aide --init...
    > Start timestamp: 2019-04-01 21:23:37 -0400 (AIDE 0.16)
    > AIDE initialized database at /var/lib/aide/aide.db.new
    > Verbose level: 6
    > 
    > Number of entries:      25973
    > 
    > ---------------------------------------------------
    > The attributes of the (uncompressed) database(s):
    > ---------------------------------------------------
    > 
    > /var/lib/aide/aide.db.new
    >   RMD160   : moyQ1YskQQbidX+Lusv3g2wf1gQ=
    >   TIGER    : 7WoOgCrXzSpDrlO6I3PyXPj1gRiaMSeo
    >   SHA256   : gVx8Fp7r3800WF2aeXl+/KHCzfGsNi7O
    >              g16VTPpIfYQ=
    >   SHA512   : GYfa0DJwWgMLl4Goo5VFVOhu4BphXCo3
    >              rZnk49PYztwu50XjaAvsVuTjJY5uIYrG
    >              tV+jt3ELvwFzGefq4ZBNMg==
    >   CRC32    : /cusZw==
    >   HAVAL    : E/i5ceF3YTjwenBfyxHEsy9Kzu35VTf7
    >              CPGQSW4tl14=
    >   GOST     : n5Ityzxey9/1jIs7LMc08SULF1sLBFUc
    >              aMv7Oby604A=
    > 
    > 
    > End timestamp: 2019-04-01 21:24:45 -0400 (run time: 1m 8s)
    > ```

1. 测试在没有任何更改时一切是否正常工作。

    在基于 Debian 的系统上：

    ``` bash
    sudo aide.wrapper --check
    ```
    
    > ```
    > Start timestamp: 2019-04-01 21:24:45 -0400 (AIDE 0.16)
    > AIDE found NO differences between database and filesystem. Looks okay!!
    > Verbose level: 6
    > 
    > Number of entries:      25973
    > 
    > ---------------------------------------------------
    > The attributes of the (uncompressed) database(s):
    > ---------------------------------------------------
    > 
    > /var/lib/aide/aide.db
    >   RMD160   : moyQ1YskQQbidX+Lusv3g2wf1gQ=
    >   TIGER    : 7WoOgCrXzSpDrlO6I3PyXPj1gRiaMSeo
    >   SHA256   : gVx8Fp7r3800WF2aeXl+/KHCzfGsNi7O
    >              g16VTPpIfYQ=
    >   SHA512   : GYfa0DJwWgMLl4Goo5VFVOhu4BphXCo3
    >              rZnk49PYztwu50XjaAvsVuTjJY5uIYrG
    >              tV+jt3ELvwFzGefq4ZBNMg==
    >   CRC32    : /cusZw==
    >   HAVAL    : E/i5ceF3YTjwenBfyxHEsy9Kzu35VTf7
    >              CPGQSW4tl14=
    >   GOST     : n5Ityzxey9/1jIs7LMc08SULF1sLBFUc
    >              aMv7Oby604A=
    > 
    > 
    > End timestamp: 2019-04-01 21:26:03 -0400 (run time: 1m 18s)
    > ```

1. 进行一些更改后，测试一切是否正常工作。

    在基于 Debian 的系统上：

    ``` bash
    sudo touch /etc/test.sh
    sudo touch /root/test.sh
    
    sudo aide.wrapper --check
    
    sudo rm /etc/test.sh
    sudo rm /root/test.sh
    
    sudo aideinit -y -f
    ```
    
    > ```
    > Start timestamp: 2019-04-01 21:37:37 -0400 (AIDE 0.16)
    > AIDE found differences between database and filesystem!!
    > Verbose level: 6
    > 
    > Summary:
    >   Total number of entries:      25972
    >   Added entries:                2
    >   Removed entries:              0
    >   Changed entries:              1
    > 
    > ---------------------------------------------------
    > Added entries:
    > ---------------------------------------------------
    > 
    > f++++++++++++++++: /etc/test.sh
    > f++++++++++++++++: /root/test.sh
    > 
    > ---------------------------------------------------
    > Changed entries:
    > ---------------------------------------------------
    > 
    > d =.... mc.. .. .: /root
    > 
    > ---------------------------------------------------
    > Detailed information about changes:
    > ---------------------------------------------------
    > 
    > Directory: /root
    >   Mtime    : 2019-04-01 21:35:07 -0400        | 2019-04-01 21:37:36 -0400
    >   Ctime    : 2019-04-01 21:35:07 -0400        | 2019-04-01 21:37:36 -0400
    > 
    > 
    > ---------------------------------------------------
    > The attributes of the (uncompressed) database(s):
    > ---------------------------------------------------
    > 
    > /var/lib/aide/aide.db
    >   RMD160   : qF9WmKaf2PptjKnhcr9z4ueCPTY=
    >   TIGER    : zMo7MvvYJcq1hzvTQLPMW7ALeFiyEqv+
    >   SHA256   : LSLLVjjV6r8vlSxlbAbbEsPcQUB48SgP
    >              pdVqEn6ZNbQ=
    >   SHA512   : Qc4U7+ZAWCcitapGhJ1IrXCLGCf1IKZl
    >              02KYL1gaZ0Fm4dc7xLqjiquWDMSEbwzW
    >              oz49NCquqGz5jpMIUy7UxA==
    >   CRC32    : z8ChEA==
    >   HAVAL    : YapzS+/cdDwLj3kHJEq8fufLp3DPKZDg
    >              U12KCSkrO7Y=
    >   GOST     : 74sLV4HkTig+GJhokvxZQm7CJD/NR0mG
    >              6jV7zdt5AXQ=
    > 
    > 
    > End timestamp: 2019-04-01 21:38:50 -0400 (run time: 1m 13s)
    > ```
    
1. 至此完成。如果将 `CRON_DAILY_RUN` 设为 `yes`（位于 `/etc/default/aide` 中），cron 就会每天执行 `/etc/cron.daily/aide`，并通过电子邮件向你发送输出。

<a id="updating-the-database"></a>
#### 更新数据库

每次更改 AIDE 监控的文件/文件夹时，都需要更新数据库以记录这些更改。在基于 Debian 的系统上，请执行：

``` bash
sudo aideinit -y -f
```

([目录](#table-of-contents))

<a id="anti-virus-scanning-with-clamav-wip"></a>
### 使用 ClamAV 进行防病毒扫描（WIP）

<a id="why-19"></a>
#### 原因

WIP

<a id="how-it-works-14"></a>
#### 工作原理

- ClamAV 是病毒扫描程序
- ClamAV-Freshclam 是用于保持病毒定义更新的服务
- ClamAV-Daemon 会让 `clamd` 进程持续运行，从而加快扫描速度

<a id="goals-18"></a>
#### 目标

WIP

<a id="notes-9"></a>
#### 注意事项

- 这些说明**不会**告诉你如何启用 ClamAV 守护进程服务以确保 `clamd` 始终运行。只有在运行邮件服务器时才需要 `clamd`，而且它不提供文件的实时监控。你应改为手动扫描文件或按计划扫描文件。

<a id="references-17"></a>
#### 参考资料

- https://www.clamav.net/documents/installation-on-debian-and-ubuntu-linux-distributions
- https://wiki.debian.org/ClamAV
- https://www.osradar.com/install-clamav-debian-9-ubuntu-18/
- https://www.lisenet.com/2014/automate-clamav-to-perform-daily-system-scan-and-send-email-notifications-on-linux/
- https://www.howtoforge.com/tutorial/configure-clamav-to-scan-and-notify-virus-and-malware/
- https://serverfault.com/questions/741299/is-there-a-way-to-keep-clamav-updated-on-debian-8
- https://askubuntu.com/questions/250290/how-do-i-scan-for-viruses-with-clamav
- https://ngothang.com/how-to-install-clamav-and-configure-daily-scanning-on-centos/

<a id="steps-19"></a>
#### 操作步骤

1. 安装 ClamAV。

    在基于 Debian 的系统上：

    ``` bash
    sudo apt install clamav clamav-freshclam clamav-daemon
    ```

1. 备份 `clamav-freshclam` 的配置文件 `/etc/clamav/freshclam.conf`：

    ``` bash
    sudo cp --archive /etc/clamav/freshclam.conf /etc/clamav/freshclam.conf-COPY-$(date +"%Y%m%d%H%M%S")
    ```
    
1. `clamav-freshclam` 的默认设置可能已经足够，但如果要更改这些设置，可以编辑文件 `/etc/clamav/freshclam.conf`，也可以使用 `dpkg-reconfigure`：

    ``` bash
    sudo dpkg-reconfigure clamav-freshclam
    ```
    
    **注意**：默认设置每天会更新定义 24 次。要更改间隔，请检查 `Checks` 设置（位于 `/etc/clamav/freshclam.conf` 中），或使用 `dpkg-reconfigure`。

1. 启动 `clamav-freshclam` 服务：

    ``` bash
    sudo service clamav-freshclam start
    ```
    
1. 可以确认 `clamav-freshclam` 是否正在运行：

    ``` bash
    sudo service clamav-freshclam status
    ```
    
    > ```
    > ● clamav-freshclam.service - ClamAV virus database updater
    >    Loaded: loaded (/lib/systemd/system/clamav-freshclam.service; enabled; vendor preset: enabled)   Active: active (running) since Sat 2019-03-16 22:57:07 EDT; 2min 13s ago
    >      Docs: man:freshclam(1)
    >            man:freshclam.conf(5)
    >            https://www.clamav.net/documents
    >  Main PID: 1288 (freshclam)
    >    CGroup: /system.slice/clamav-freshclam.service
    >            └─1288 /usr/bin/freshclam -d --foreground=true
    > 
    > Mar 16 22:57:08 host freshclam[1288]: Sat Mar 16 22:57:08 2019 -> ^Local version: 0.100.2 Recommended version: 0.101.1
    > Mar 16 22:57:08 host freshclam[1288]: Sat Mar 16 22:57:08 2019 -> DON'T PANIC! Read https://www.clamav.net/documents/upgrading-clamav
    > Mar 16 22:57:15 host freshclam[1288]: Sat Mar 16 22:57:15 2019 -> Downloading main.cvd [100%]
    > Mar 16 22:57:38 host freshclam[1288]: Sat Mar 16 22:57:38 2019 -> main.cvd updated (version: 58, sigs: 4566249, f-level: 60, builder: sigmgr)
    > Mar 16 22:57:40 host freshclam[1288]: Sat Mar 16 22:57:40 2019 -> Downloading daily.cvd [100%]
    > Mar 16 22:58:13 host freshclam[1288]: Sat Mar 16 22:58:13 2019 -> daily.cvd updated (version: 25390, sigs: 1520006, f-level: 63, builder: raynman)
    > Mar 16 22:58:14 host freshclam[1288]: Sat Mar 16 22:58:14 2019 -> Downloading bytecode.cvd [100%]
    > Mar 16 22:58:16 host freshclam[1288]: Sat Mar 16 22:58:16 2019 -> bytecode.cvd updated (version: 328, sigs: 94, f-level: 63, builder: neo)
    > Mar 16 22:58:24 host freshclam[1288]: Sat Mar 16 22:58:24 2019 -> Database updated (6086349 signatures) from db.local.clamav.net (IP: 104.16.219.84)
    > Mar 16 22:58:24 host freshclam[1288]: Sat Mar 16 22:58:24 2019 -> ^Clamd was NOT notified: Can't connect to clamd through /var/run/clamav/clamd.ctl: No such file or directory
    > ```
    
    **注意**：不必担心 `Local version` 那一行。有关更多详细信息，请查看 https://serverfault.com/questions/741299/is-there-a-way-to-keep-clamav-updated-on-debian-8 。

1. 备份 `clamav-daemon` 的配置文件 `/etc/clamav/clamd.conf`：

    ``` bash
    sudo cp --archive /etc/clamav/clamd.conf /etc/clamav/clamd.conf-COPY-$(date +"%Y%m%d%H%M%S")
    ```
    
1. 可以通过编辑文件来更改 `clamav-daemon` 的设置；请编辑 `/etc/clamav/clamd.conf` 或使用 `dpkg-reconfigure`：

    ``` bash
    sudo dpkg-reconfigure clamav-daemon
    ```

<a id="scanning-filesfolders"></a>
#### 扫描文件/文件夹

- 使用 `clamscan` 程序扫描文件/文件夹。
- `clamscan` 会以执行它的用户身份运行，因此该用户需要对所扫描的文件/文件夹拥有读取权限。
- 使用 `clamscan` 时采用 `root` 身份很危险，因为如果某个文件确实是病毒，它可能会利用 root 权限。
- 扫描文件：`clamscan /path/to/file`。
- 扫描目录：`clamscan -r /path/to/folder`。
- 可以使用 `-i` 开关，只输出受感染的文件。
- 有关其他开关/选项，请查看 `clamscan` 的 `man` 页面。

([目录](#table-of-contents))

<a id="rootkit-detection-with-rkhunter-wip"></a>
### 使用 Rkhunter 检测 Rootkit（WIP）

<a id="why-20"></a>
#### 原因

WIP

<a id="how-it-works-15"></a>
#### 工作原理

WIP

<a id="goals-19"></a>
#### 目标

WIP

<a id="references-18"></a>
#### 参考资料

- http://rkhunter.sourceforge.net/
- https://www.cyberciti.biz/faq/howto-check-linux-rootkist-with-detectors-software/
- https://www.tecmint.com/install-rootkit-hunter-scan-for-rootkits-backdoors-in-linux/

<a id="steps-20"></a>
#### 操作步骤

1. 安装 Rkhunter。

    在基于 Debian 的系统上：
    
    ``` bash
    sudo apt install rkhunter
    ```

1. 备份 rkhunter 的默认设置文件：

    ``` bash
    sudo cp -p /etc/default/rkhunter /etc/default/rkhunter-COPY-$(date +"%Y%m%d%H%M%S")
    ```

1. rkhunter 的配置文件是 `/etc/rkhunter.conf`。不要修改该文件，而应创建并使用文件 `/etc/rkhunter.conf.local`：

    ``` bash
    sudo cp -p /etc/rkhunter.conf /etc/rkhunter.conf.local
    ```
    
1. 检查配置文件 `/etc/rkhunter.conf.local`，并根据自己的需求进行设置。我的建议如下：

    |设置|备注|
    |--|--|
    |`UPDATE_MIRRORS=1`||
    |`MIRRORS_MODE=0`||
    |`MAIL-ON-WARNING=root`||
    |`COPY_LOG_ON_ERROR=1`|发生错误时保存日志副本|
    |`PKGMGR=...`|根据文档设置为适当的值|
    |`PHALANX2_DIRTEST=1`|阅读文档以了解原因|
    |`WEB_CMD=""`|用于解决 Debian 软件包禁用 rkhunter 自我更新能力的问题。|
    |`USE_LOCKING=1`|防止 rkhunter 多次运行导致问题|
    |`SHOW_SUMMARY_WARNINGS_NUMBER=1`|查看实际发现的警告数量|

1. 你希望 rkhunter 每天运行并通过电子邮件发送结果。可以编写自己的脚本，也可以查看 https://www.tecmint.com/install-rootkit-hunter-scan-for-rootkits-backdoors-in-linux/ ，获取可供使用的 cron 示例脚本。
   
    在基于 Debian 的系统上，rkhunter 随附 cron 脚本。要启用这些脚本，请检查 `/etc/default/rkhunter`，或使用 `dpkg-reconfigure` 并对所有问题回答 `Yes`：
    
    ``` bash
    sudo dpkg-reconfigure rkhunter
    ```

1. 完成所有更改后，确保所有设置都有效：

    ``` bash
    sudo rkhunter -C
    ```

1. 更新 rkhunter 及其数据库：

    ``` bash
    sudo rkhunter --versioncheck
    sudo rkhunter --update
    sudo rkhunter --propupd
    ```

1. 如果要执行手动扫描并查看输出：

    ``` bash
    sudo rkhunter --check
    ```

([目录](#table-of-contents))

<a id="rootkit-detection-with-chrootkit-wip"></a>
### 使用 chrootkit 检测 Rootkit（WIP）

<a id="why-21"></a>
#### 原因

WIP

<a id="how-it-works-16"></a>
#### 工作原理

WIP

<a id="goals-20"></a>
#### 目标

WIP

<a id="references-19"></a>
#### 参考资料

- http://www.chkrootkit.org/
- https://www.cyberciti.biz/faq/howto-check-linux-rootkist-with-detectors-software/
- https://askubuntu.com/questions/258658/eth0-packet-sniffer-sbin-dhclient

<a id="steps-21"></a>
#### 操作步骤

1. 安装 chkrootkit。

    在基于 Debian 的系统上：
    
    ``` bash
    sudo apt install chkrootkit
    ```

1. 执行手动扫描：

    ``` bash
    sudo chkrootkit
    ```
    
    > ```
    > ROOTDIR is `/'
    > Checking `amd'...                                           not found
    > Checking `basename'...                                      not infected
    > Checking `biff'...                                          not found
    > Checking `chfn'...                                          not infected
    > Checking `chsh'...                                          not infected
    > ...
    > Checking `scalper'...                                       not infected
    > Checking `slapper'...                                       not infected
    > Checking `z2'...                                            chklastlog: nothing deleted
    > Checking `chkutmp'...                                       chkutmp: nothing deleted
    > Checking `OSX_RSPLUG'...                                    not infected
    > ```

1. 备份 chkrootkit 的配置文件 `/etc/chkrootkit.conf`：

    ``` bash
    sudo cp --archive /etc/chkrootkit.conf /etc/chkrootkit.conf-COPY-$(date +"%Y%m%d%H%M%S")
    ```

1. 你希望 chkrootkit 每天运行并通过电子邮件发送结果。
   
    在基于 Debian 的系统上，chkrootkit 随附 cron 脚本。要启用这些脚本，请检查 `/etc/chkrootkit.conf`，或使用 `dpkg-reconfigure` 并对第一个问题回答 `Yes`：
    
    ``` bash
    sudo dpkg-reconfigure chkrootkit
    ```

([目录](#table-of-contents))

<a id="logwatch---system-log-analyzer-and-reporter"></a>
### logwatch - 系统日志分析与报告工具

<a id="why-22"></a>
#### 原因

服务器会生成大量日志，其中可能包含重要信息。除非打算每天检查服务器，否则你会需要一种通过电子邮件获取服务器日志摘要的方法。为此，我们将使用 [logwatch](https://sourceforge.net/projects/logwatch/)。

<a id="how-it-works-17"></a>
#### 工作原理

logwatch 会扫描系统日志文件并对其进行汇总。可以直接从命令行运行它，也可以安排它按周期运行。logwatch 使用服务文件来了解如何读取/汇总日志文件。可以在 `/usr/share/logwatch/scripts/services` 中查看所有随附的默认服务文件。

logwatch 的配置文件 `/usr/share/logwatch/default.conf/logwatch.conf` 指定默认选项。可以通过命令行参数覆盖这些选项。

<a id="goals-21"></a>
#### 目标

- 将 Logwatch 配置为每天通过电子邮件发送服务器所有状态和日志的摘要

<a id="notes-10"></a>
#### 注意事项

- 要使此功能正常工作，服务器必须能够发送电子邮件
- 按照以下步骤操作后，logwatch 将每天运行。如果要更改计划，请按需修改 cron 作业。还应更改 `range` 选项，使其覆盖你的重复执行时间窗口。示例请参阅 https://www.badpenguin.org/configure-logwatch-for-weekly-email-and-html-output-format 。
- 如果由于电子邮件中存在过长的行而导致 logwatch 无法投递邮件，请查看 https://blog.dhampir.no/content/exim4-line-length-in-debian-stretch-mail-delivery-failed-returning-message-to-sender ，如[问题 #29](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/issues/29)中所记录。如果你遵循了[使用隐式 TLS 将 Gmail 和 Exim4 配置为 MTA](#gmail-and-exim4-as-mta-with-implicit-tls)，我们已经在第 7 步中处理了此问题。

<a id="references-20"></a>
#### 参考资料

- 感谢 [amacheema](https://github.com/amacheema) 修复步骤中的一些问题，并告知我 [问题 #29](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/issues/29) 中记录的 exim4 长行错误。
- https://sourceforge.net/projects/logwatch/
- https://www.digitalocean.com/community/tutorials/how-to-install-and-use-logwatch-log-analyzer-and-reporter-on-a-vps

<a id="steps-22"></a>
#### 操作步骤

1. 安装 logwatch。

    在基于 Debian 的系统上：

    ``` bash
    sudo apt install logwatch
    ```

1. 要查看 logwatch 所收集内容的示例，可以直接运行它：

    ``` bash
    sudo /usr/sbin/logwatch --output stdout --format text --range yesterday --service all
    ```

    > ```
    > 
    >  ################### Logwatch 7.4.3 (12/07/16) ####################
    >         Processing Initiated: Mon Mar  4 00:05:50 2019
    >         Date Range Processed: yesterday
    >                               ( 2019-Mar-03 )
    >                               Period is day.
    >         Detail Level of Output: 5
    >         Type of Output/Format: stdout / text
    >         Logfiles for Host: host
    >  ##################################################################
    > 
    >  --------------------- Cron Begin ------------------------
    > ...
    > ...
    >  ---------------------- Disk Space End -------------------------
    > 
    > 
    >  ###################### Logwatch End #########################
    > ```

1. 继续之前，请通读 logwatch 带有自说明的配置文件 `/usr/share/logwatch/default.conf/logwatch.conf`。这里无需更改任何内容，但请特别留意 `Output`、`Format`、`MailTo`、`Range` 和 `Service`，因为我们将使用这些选项。就我们的用途而言，不在配置文件中指定选项，而是将它们作为命令行参数传递给每天执行 logwatch 的 cron 作业。这样，即使配置文件日后被修改（例如在更新期间），我们的选项仍会保留。

1. 备份 logwatch 的每日 cron 文件 `/etc/cron.daily/00logwatch`，并取消其执行位：

    ``` bash
    sudo cp --archive /etc/cron.daily/00logwatch /etc/cron.daily/00logwatch-COPY-$(date +"%Y%m%d%H%M%S")
    sudo chmod -x /etc/cron.daily/00logwatch-COPY*
    ```

1. 默认情况下，logwatch 会输出到 `stdout`。由于目标是每天收到电子邮件，因此需要更改 logwatch 使用的输出类型，改为发送电子邮件。可以通过上面的配置文件完成此操作，但那样会应用于每一次运行——即使我们手动运行并希望在屏幕上查看输出时也会如此。我们改为修改执行 logwatch 的 cron 作业，让它发送电子邮件。这样，手动运行时仍会获得输出到 `stdout` 的内容，而由 cron 运行时则会发送电子邮件。我们还会确保它检查所有服务，并将输出格式改为 html，以便无论配置文件如何设置都更易于阅读。在文件 `/etc/cron.daily/00logwatch` 中找到执行行，并将其改为：

    ```
    /usr/sbin/logwatch --output mail --format html --mailto root --range yesterday --service all
    ```

    > ```
    > #!/bin/bash
    > 
    > #Check if removed-but-not-purged
    > test -x /usr/share/logwatch/scripts/logwatch.pl || exit 0
    > 
    > #execute
    > /usr/sbin/logwatch --output mail --format html --mailto root --range yesterday --service all
    > 
    > #Note: It's possible to force the recipient in above command
    > #Just pass --mailto address@a.com instead of --output mail
    > ```

    [懒人版](#editing-configuration-files---for-the-lazy)：
    
    ``` bash
    sudo sed -i -r -e "s,^($(sudo which logwatch).*?),# \1         # commented by $(whoami) on $(date +"%Y-%m-%d @ %H:%M:%S")\n$(sudo which logwatch) --output mail --format html --mailto root --range yesterday --service all         # added by $(whoami) on $(date +"%Y-%m-%d @ %H:%M:%S")," /etc/cron.daily/00logwatch
    ```

1. 可以通过执行该 cron 作业来测试它：

    ``` bash
    sudo /etc/cron.daily/00logwatch
    ```
    
    **注意**：如果由于电子邮件中存在过长的行而导致 logwatch 无法投递邮件，请查看 https://blog.dhampir.no/content/exim4-line-length-in-debian-stretch-mail-delivery-failed-returning-message-to-sender ，如[问题 #29](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/issues/29)中所记录。如果你遵循了[使用隐式 TLS 将 Gmail 和 Exim4 配置为 MTA](#gmail-and-exim4-as-mta-with-implicit-tls)，我们已经在第 7 步中处理了此问题。

([目录](#table-of-contents))

<a id="ss---seeing-ports-your-server-is-listening-on"></a>
### ss - 查看服务器正在侦听的端口

<a id="why-23"></a>
#### 原因

端口是应用程序、服务和进程相互通信的方式——既可用于服务器内部的本地通信，也可用于与网络上的其他设备通信。当服务器上运行应用程序或服务（例如 SSH 或 Apache）时，它们会在特定端口上侦听请求。

显然，我们不希望服务器在我们不知情的端口上侦听。我们将使用 `ss` 查看服务正在侦听的所有端口。这有助于追查并停止未经授权且可能危险的服务。

<a id="goals-22"></a>
#### 目标

- 查明除 localhost 以外有哪些端口已开放并正在侦听连接

<a id="references-21"></a>
#### 参考资料

- https://www.reddit.com/r/linux/comments/arx7st/howtosecurealinuxserver_an_evolving_howto_guide/egrib6o/
- https://www.reddit.com/r/linux/comments/arx7st/howtosecurealinuxserver_an_evolving_howto_guide/egs1rev/
- https://www.tecmint.com/find-open-ports-in-linux/
- `man ss`

<a id="steps-23"></a>
#### 操作步骤

1. 查看正在侦听流量的所有端口：

    ``` bash
    sudo ss -lntup
    ```
    
    > ```
    > Netid  State      Recv-Q Send-Q     Local Address:Port     Peer Address:Port
    > udp    UNCONN     0      0                      *:68                  *:*        users:(("dhclient",pid=389,fd=6))
    > tcp    LISTEN     0      128                    *:22                  *:*        users:(("sshd",pid=4390,fd=3))
    > tcp    LISTEN     0      128                   :::22                 :::*        users:(("sshd",pid=4390,fd=4))
    > ```
    
    **开关说明**：
    - `l` = 显示正在侦听的套接字
    - `n` = 不尝试解析服务名称
    - `t` = 显示 TCP 套接字
    - `u` = 显示 UDP 套接字
    - `p` = 显示进程信息

1. 如果看到任何可疑内容，例如不知情的端口或不认识的进程，请进行调查并视需要采取补救措施。

([目录](#table-of-contents))

<a id="lynis---linux-security-auditing"></a>
### Lynis - Linux 安全审计

<a id="why-24"></a>
#### 原因

摘自 [https://cisofy.com/lynis/](https://cisofy.com/lynis/)：

> Lynis 是一款久经考验的安全工具，适用于运行 Linux、macOS 或基于 Unix 的操作系统的系统。它会对系统执行广泛的运行状况扫描，以支持系统加固和合规性测试。

<a id="goals-23"></a>
#### 目标

- 已安装 Lynis

<a id="notes-11"></a>
#### 注意事项

- CISOFY 为许多发行版提供软件包。有关特定发行版的安装说明，请查看 https://packages.cisofy.com/ 。

<a id="references-22"></a>
#### 参考资料

- https://cisofy.com/documentation/lynis/get-started/
- https://packages.cisofy.com/community/#debian-ubuntu
- https://thelinuxcode.com/audit-lynis-ubuntu-server/
- https://www.vultr.com/docs/install-lynis-on-debian-8

<a id="steps-24"></a>
#### 操作步骤

1. 安装 lynis。https://cisofy.com/lynis/#installation 提供了针对你的发行版安装 lynis 的详细说明。

    在基于 Debian 的系统上，使用 CISOFY 的社区软件仓库：

    ``` bash
	sudo apt install ca-certificates host
	sudo mkdir -p /etc/apt/keyrings
	wget -O - https://packages.cisofy.com/keys/cisofy-software-public.key | sudo gpg --dearmor -o /etc/apt/keyrings/cisofy-lynis.gpg
	echo "deb [signed-by=/etc/apt/keyrings/cisofy-lynis.gpg] https://packages.cisofy.com/community/lynis/deb/ stable main" | sudo tee /etc/apt/sources.list.d/cisofy-lynis.list
	sudo apt update
	sudo apt install lynis
    ```

1. 更新 lynis：

    ``` bash
    sudo lynis update info
    ```

1. 运行安全审计：

    ``` bash
    sudo lynis audit system
    ```

    这会扫描服务器、报告审计发现，并在最后给出建议。请花些时间检查输出，并视需要弥补不足。

([目录](#table-of-contents))

<a id="ossec---host-intrusion-detection"></a>
### OSSEC - 主机入侵检测

<a id="why-25"></a>
#### 原因
摘自 [https://github.com/ossec/ossec-hids](https://github.com/ossec/ossec-hids)
> OSSEC 是一个用于监控和控制系统的完整平台。它将 HIDS（基于主机的入侵检测）、日志监控和 SIM/SIEM 的所有方面融合为一个简单、强大且开源的解决方案。

<a id="goals-24"></a>
#### 目标

- 已安装 OSSEC-HIDS

<a id="references-23"></a>
#### 参考资料

- https://www.ossec.net/docs/

<a id="steps-25"></a>
#### 操作步骤

1. 从源代码安装 OSSEC-HIDS
    ```bash
    sudo apt install -y libz-dev libssl-dev libpcre2-dev build-essential libsystemd-dev
    wget https://github.com/ossec/ossec-hids/archive/3.7.0.tar.gz
    tar xzf 3.7.0.tar.gz
    cd ossec-hids-3.7.0/
    sudo ./install.sh
    ```

1. 实用命令：

**Agent 信息**

   ```bash
    sudo /var/ossec/bin/agent_control -i <AGENT_ID>
   ```
`AGENT_ID` 默认为 `000`；可以使用命令 `sudo /var/ossec/bin/agent_control -l` 进行确认。

**运行完整性/rootkit 检查**

默认情况下，OSSEC 每 2 小时运行一次 rootkit 检查。

   ```bash
    sudo /var/ossec/bin/agent_control -u <AGENT_ID> -r 
   ```

**警报**

- 全部：
    ```bash
    tail -f /var/ossec/logs/alerts/alerts.log
    ```
- 完整性检查：
    ```bash
    sudo cat /var/ossec/logs/alerts/alerts.log | grep -A4  -i integrity
    ```
- Rootkit 检查：
    ```bash
     sudo cat /var/ossec/logs/alerts/alerts.log | grep -A4  "rootcheck,"
    ```

([目录](#table-of-contents))
<a id="the-danger-zone"></a>
## 高风险操作区

### 风险自负，谨慎操作

本节涵盖高风险操作，因为这些操作有可能导致系统无法使用；此外，由于风险大于任何收益，许多人认为这些操作没有必要。

**!! 风险自负，谨慎操作 !!**

<details><summary>!! 风险自负，谨慎操作 !!</summary>

([目录](#table-of-contents))

### 目录

- [Linux 内核 sysctl 加固](#linux-kernel-sysctl-hardening)
- [为 GRUB 设置密码保护](#password-protect-grub)
- [禁用 root 登录](#disable-root-login)
- [更改默认 umask](#change-default-umask)
- [孤立软件](#orphaned-software)

([目录](#table-of-contents))

<a id="linux-kernel-sysctl-hardening"></a>
### Linux 内核 sysctl 加固

<details><summary>!! 风险自负，谨慎操作 !!</summary>

#### 原因

内核是 Linux 系统的大脑。保护内核的安全合情合理。

#### 不采用的理由

使用 sysctl 更改内核设置存在风险，并且可能会破坏服务器。如果你不知道自己在做什么、没有时间调试问题，或者只是不想承担风险，我建议不要执行这些步骤。

#### 免责声明

我对 Linux 内核加固和保护方面的了解没有自己期望的那么多。尽管我不愿承认，但我并不知道所有这些设置的作用。我的理解是，其中大多数属于常规内核加固和性能设置，其余设置用于防范欺骗和 DOS 攻击。

事实上，由于我并不能百分之百确定每项设置的确切作用，因此我采用了许多网站推荐的设置（下面的参考资料中均有链接），并将其汇总起来，以确定应当设置哪些内容。我认为，如果多个信誉良好的网站都提到同一项设置，那么它很可能是安全的。

如果你更了解这些设置的作用，或者有任何其他反馈或建议，请[告诉我](#contacting-me)。

本节不会提供[懒人版](#editing-configuration-files---for-the-lazy)代码。

#### 注意事项

- 所有 sysctl 设置和键的文档都严重不足。我[能找到的文档](https://github.com/torvalds/linux/tree/master/Documentation)似乎引用的是 2.2 版内核。我找不到任何更新的文档。如果你知道在哪里可以找到，请[告诉我](#contacting-me)。
- 下面列出的参考网站对每项设置的作用提供了更多说明。

#### 参考资料

- https://github.com/torvalds/linux/tree/master/Documentation
- https://www.cyberciti.biz/faq/linux-kernel-etcsysctl-conf-security-hardening/
- https://geektnt.com/sysctl-conf-hardening.html
- https://linoxide.com/how-tos/linux-server-protection/
- https://github.com/klaver/sysctl/blob/master/sysctl.conf
- https://cloudpro.zone/index.php/2018/01/30/debian-9-3-server-setup-guide-part-5/

#### 操作步骤

1. sysctl 设置可以在本仓库的 [linux-kernel-sysctl-hardening.md](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/blob/master/linux-kernel-sysctl-hardening.md) 文件中找到。

1. 在将内核 sysctl 更改永久化之前，可以使用 sysctl 命令对其进行测试：

    ``` bash
    sudo sysctl -w [key=value]
    ```

    示例：

    ``` bash
    sudo sysctl -w kernel.ctrl-alt-del=0
    ```

    **注意**：`key=value` 中不能有空格，等号前后也不能有空格。

1. 测试某项设置并确保它不会破坏服务器之后，可以通过将值添加到 `/etc/sysctl.conf` 来使其永久生效。例如：

    ``` bash
    $ sudo cat /etc/sysctl.conf
    kernel.ctrl-alt-del = 0
    fs.file-max = 65535
    ...
    kernel.sysrq = 0
    ```

1. 更新文件后，可以重新加载设置或重启。重新加载：

    ``` bash
    sudo sysctl -p
    ```

**注意**：如果 sysctl 在写入任何设置时遇到问题，`sysctl -w` 或 `sysctl -p` 会将错误写入 stderr。可以利用这一点快速找到 `/etc/sysctl.conf` 文件中的无效设置：

``` bash
sudo sysctl -p >/dev/null
```

</details><br />

([目录](#table-of-contents))

<a id="password-protect-grub"></a>
### 为 GRUB 设置密码保护

<details><summary>!! 风险自负，谨慎操作 !!</summary>

#### 原因

如果恶意行为者能够物理接触你的服务器，他们可能会利用 GRUB 未经授权地访问你的系统。

#### 不采用的理由

如果忘记密码，则必须经过[一些操作](https://www.cyberciti.biz/tips/howto-recovering-grub-boot-loader-password.html)才能恢复密码。

#### 目标

- 自动启动默认的 Debian 安装，其他任何操作都需要密码

#### 注意事项

- 这只能保护 GRUB 以及它之后的内容，例如操作系统。请查看主板文档，了解如何为 BIOS 设置密码保护，以防止恶意行为者绕过 GRUB。

#### 参考资料

- https://selivan.github.io/2017/12/21/grub2-password-for-all-but-default-menu-entries.html
- https://help.ubuntu.com/community/Grub2/Passwords
- https://computingforgeeks.com/how-to-protect-grub-with-password-on-debian-ubuntu-and-kali-linux/
- `man grub`
- `man grub-mkpasswd-pbkdf2`

#### 操作步骤

1. 为密码创建一个[基于密码的密钥派生函数 2（PBKDF2）](https://en.wikipedia.org/wiki/PBKDF2)哈希：

    ``` bash
    grub-mkpasswd-pbkdf2 -c 100000
    ```

   以下输出使用 `password` 作为密码：

    > ```
    > Enter password:
    > Reenter password:
    > PBKDF2 hash of your password is grub.pbkdf2.sha512.100000.2812C233DFC899EFC3D5991D8CA74068C99D6D786A54F603E9A1EFE7BAEDDB6AA89672F92589FAF98DB9364143E7A1156C9936328971A02A483A84C3D028C4FF.C255442F9C98E1F3C500C373FE195DCF16C56EEBDC55ABDD332DD36A92865FA8FC4C90433757D743776AB186BD3AE5580F63EF445472CC1D151FA03906D08A6D
    > ```

1. 复制 `PBKDF2 hash of your password is ` **之后**的所有内容，**从 `grub.pbkdf2.sha512...` 开始并包括它**，一直复制到末尾。下一步需要用到这些内容。

1. `update-grub` 程序使用脚本生成它将用于 GRUB 设置的配置文件。创建文件 `/etc/grub.d/01_password`，将下面的代码添加进去，并将 `[hash]` 替换为第一步中复制的哈希。这会让 `update-grub` 为 GRUB 使用此用户名和密码。

    ``` bash
    #!/bin/sh
    set -e

    cat << EOF
    set superusers="grub"
    password_pbkdf2 grub [hash]
    EOF
    ```

    例如：

    > ``` bash
    > #!/bin/sh
    > set -e
    > 
    > cat << EOF
    > set superusers="grub"
    > password_pbkdf2 grub grub.pbkdf2.sha512.100000.2812C233DFC899EFC3D5991D8CA74068C99D6D786A54F603E9A1EFE7BAEDDB6AA89672F92589FAF98DB9364143E7A1156C9936328971A02A483A84C3D028C4FF.C255442F9C98E1F3C500C373FE195DCF16C56EEBDC55ABDD332DD36A92865FA8FC4C90433757D743776AB186BD3AE5580F63EF445472CC1D151FA03906D08A6D
    > EOF
    > ```

1. 设置文件的执行位，以便 `update-grub` 在更新 GRUB 配置时包含该文件：

   ``` bash
   sudo chmod a+x /etc/grub.d/01_password
   ```

1. 备份即将修改的 GRUB 配置文件 `/etc/grub.d/10_linux`，并取消其副本的执行位，以免 `update-grub` 尝试运行它们：

    ``` bash
    sudo cp --archive /etc/grub.d/10_linux /etc/grub.d/10_linux-COPY-$(date +"%Y%m%d%H%M%S")
    sudo chmod a-x /etc/grub.d/10_linux.*
    ```

1. 要使默认的 Debian 安装不受限制（**无需**密码），同时让其他所有内容都受限制（**需要**密码），请修改 `/etc/grub.d/10_linux`，并将 `--unrestricted` 添加到 `CLASS` 变量。

    [懒人版](#editing-configuration-files---for-the-lazy)：

    ``` bash
    sudo sed -i -r -e "/^CLASS=/ a CLASS=\"\${CLASS} --unrestricted\"         # added by $(whoami) on $(date +"%Y-%m-%d @ %H:%M:%S")" /etc/grub.d/10_linux
    ```

1. 使用 `update-grub` 更新 GRUB：

    ``` bash
    sudo update-grub
    ```

</details><br />

([目录](#table-of-contents))

<a id="disable-root-login"></a>
### 禁用 root 登录

<details><summary>!! 风险自负，谨慎操作 !!</summary>

#### 原因

如果 sudo 已[正确配置](#limit-who-can-use-sudo)，那么 **root** 账户几乎永远不需要直接登录，无论是在终端还是远程登录。

#### 不采用的理由

**警告：这可能会导致某些配置出现问题！**

如果你的安装使用 [`sulogin`](https://linux.die.net/man/8/sulogin)（例如 Debian）在启动失败时进入 **root** 控制台，那么锁定 **root** 账户将阻止 `sulogin` 打开 **root** shell，并且会出现以下错误：

    Cannot open access to console, the root account is locked.
    
    See sulogin(8) man page for more details.
    
    Press Enter to continue.

要解决此问题，可以使用 `--force` 选项来运行 `sulogin`。有些发行版已包含此解决方法或其他解决方法。

锁定 **root** 账户的另一种做法是设置一个很长且复杂的 **root** 密码，并将其保存在安全的非数字化介质中。这样，在需要时就能取用该密码。

#### 目标

- 锁定 **root** 账户，使任何人都无法以 **root** 身份登录

#### 注意事项

- 某些发行版默认禁用 **root** 登录（例如 Ubuntu），因此可能不需要执行此步骤。请查阅所用发行版的文档。

#### 参考资料

- https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=806852
- https://github.com/systemd/systemd/issues/7115
- https://github.com/karelzak/util-linux/commit/7ff1162e67164cb4ece19dd809c26272461aa254
- https://github.com/systemd/systemd/issues/11596
- https://www.reddit.com/r/selfhosted/comments/aoxd4l/new_guide_created_by_me_how_to_secure_a_linux/eg4rkfi/
- `man systemd`

#### 操作步骤

1. 锁定 **root** 账户：

    ``` bash
    sudo passwd -l root
    ```

</details><br />

([目录](#table-of-contents))

<a id="change-default-umask"></a>
### 更改默认 umask

<details><summary>!! 风险自负，谨慎操作 !!</summary>

#### 原因

umask 控制文件和文件夹创建时的**默认**权限。不安全的文件或文件夹权限可能会让其他账户未经授权访问你的数据。这可能包括进行配置更改的能力。

- 对于**非 root** 账户，**默认情况下**无需让其他账户获得该账户文件或文件夹的任何访问权限。
- 对于 **root** 账户，**默认情况下**无需让文件或文件夹的主组或其他账户获得 **root** 文件或文件夹的任何访问权限。

当其他账户确实需要访问某个文件或文件夹时，应使用文件或文件夹权限与主组的组合显式授予访问权限。

#### 不采用的理由

更改默认 umask 可能会产生意外问题。例如，如果将 **root** 的 umask 设置为 `0077`，那么**非 root** 账户将**无法**访问 `/etc/` 中的应用程序配置文件或文件夹，这可能会破坏不以 **root** 权限运行的应用程序。

#### 工作原理

要解释 umask 的工作原理，我必须先解释 Linux 文件和文件夹权限的工作原理。由于这是一个相当复杂的问题，我建议你阅读下面的参考资料以进一步了解。

#### 目标

- 将**非 root** 账户的默认 umask 设置为 **0027**
- 将 **root** 账户的默认 umask 设置为 **0077**

#### 注意事项

- umask 是 Bash 内置命令，这意味着用户可以更改自己的 umask 设置。

#### 参考资料

- https://www.linuxnix.com/umask-define-linuxunix/
- https://serverfault.com/questions/818783/which-umask-is-more-secure-in-linux-022-or-027
- https://www.cyberciti.biz/tips/understanding-linux-unix-umask-value-usage.html
- `man umask`

#### 操作步骤

1. 备份将要编辑的文件：

    ``` bash
    sudo cp --archive /etc/profile /etc/profile-COPY-$(date +"%Y%m%d%H%M%S")
    sudo cp --archive /etc/bash.bashrc /etc/bash.bashrc-COPY-$(date +"%Y%m%d%H%M%S")
    sudo cp --archive /etc/login.defs /etc/login.defs-COPY-$(date +"%Y%m%d%H%M%S")
    sudo cp --archive /root/.bashrc /root/.bashrc-COPY-$(date +"%Y%m%d%H%M%S")
    ```

1. 将以下行添加到 `/etc/profile` 和 `/etc/bash.bashrc`，把**非 root** 账户的默认 umask 设置为 **0027**：

    ```
    umask 0027
    ```

    [懒人版](#editing-configuration-files---for-the-lazy)：

    ``` bash
    echo -e "\numask 0027         # added by $(whoami) on $(date +"%Y-%m-%d @ %H:%M:%S")" | sudo tee -a /etc/profile /etc/bash.bashrc
    ```

1. 还需要将以下行添加到 `/etc/login.defs`：

    ```
    UMASK 0027
    ```

    [懒人版](#editing-configuration-files---for-the-lazy)：

    ``` bash
    echo -e "\nUMASK 0027         # added by $(whoami) on $(date +"%Y-%m-%d @ %H:%M:%S")" | sudo tee -a /etc/login.defs
    ```

1. 将以下行添加到 `/root/.bashrc`，把 **root** 账户的默认 umask 设置为 **0077**：

    ```
    umask 0077
    ```

    [懒人版](#editing-configuration-files---for-the-lazy)：

    ``` bash
    echo -e "\numask 0077         # added by $(whoami) on $(date +"%Y-%m-%d @ %H:%M:%S")" | sudo tee -a /root/.bashrc
    ```

</details><br />

([目录](#table-of-contents))

<a id="orphaned-software"></a>
### 孤立软件

<details><summary>!! 风险自负，谨慎操作 !!</summary>

#### 原因

随着你使用系统并安装和卸载软件，最终会留下孤立或未使用的软件、软件包和库。你并非必须删除它们，但如果不需要，为什么还要保留？当安全是首要任务时，任何并非明确需要的内容都是潜在的安全威胁。你应尽可能让服务器保持精简。

#### 注意事项

- 每个发行版管理软件、软件包和库的方式不同，因此查找和删除孤立软件包的方法也会不同。目前我只有适用于 Debian 系统的步骤。

#### 基于 Debian 的系统

在基于 Debian 的系统上，可以使用 [deborphan](http://freshmeat.sourceforge.net/projects/deborphan/) 查找孤立软件包。

##### <a name="orphaned-software-why-not"></a>不采用的理由

请记住，deborphan 查找的是**不存在软件包依赖关系**的软件包。这并不意味着它们未被使用。你完全可能每天都在使用某个没有依赖项、因而不应删除的软件包。此外，如果 deborphan 判断错误，删除关键软件包可能会破坏系统。

##### 操作步骤

1. 安装 deborphan。

    ``` bash
    sudo apt install deborphan
    ```

1. 以 **root** 身份运行 deborphan，查看孤立软件包列表：

    ``` bash
    sudo deborphan
    ```

    > ```
    > libxapian30
    > libpipeline1
    > ```

1. [假设你确实要删除 deborphan 找到的所有软件包](#orphaned-software-why-not)，可以将它的输出传递给 `apt` 以删除这些软件包：

    ``` bash
    sudo apt --autoremove purge $(deborphan)
    ```

</details>

</details><br />

([目录](#table-of-contents))

<a id="the-miscellaneous"></a>
## 其他内容

<a id="the-simple-way-with-msmtp"></a>
### 使用 MSMTP 的简单方法
#### 原因

我会将此方法**简化**为仅使用 Google Mail 账户（以及其他账户）发送电子邮件。真正的简单！:)

    ``` bash
    #!/bin/bash
    ###### PLEASE .... EDIT IT...
    USEREMAIL="usernameemail"
    DOMPROV="gmail.com"
    PWDEMAIL="passwordStrong"  ## ATTENTION DONT USE Special Chars.. like as SPACE # and some others not all. Feel free to test ;)
    MAILPROV="smtp.google.com:583"
    MYMAIL="$USRMAIL@$DOMPROV"
    USERLOC="root"
    #######
    apt install -y msmtp
        ln -s /usr/bin/msmtp /usr/sbin/sendmail
    #wget http://www.cacert.org/revoke.crl -O /etc/ssl/certs/revoke.crl
    #chmod 644 /etc/ssl/certs/revoke.crl
    touch /root/.msmtprc
    cat <<EOF> .msmtprc
    defaults
    account gmail
    host $MAILPROV
    port $MAILPORT
    #proxy_host 127.0.0.1
    #proxy_port 9001
    from $MYEMAIL
    timeout off 
    protocol smtp
    #auto_from [(on|off)]
    #from envelope_from
    #maildomain [domain]
    auth on
    user $USRMAIL
    passwordeval "gpg -q --for-your-eyes-only --no-tty -d /root/msmtp-mail.gpg"
    #passwordeval "gpg --quiet --for-your-eyes-only --no-tty --decrypt /root/msmtp-mail.gpg"
    tls on
    tls_starttls on
    tls_trust_file /etc/ssl/certs/ca-certificates.crt
    #tls_crl_file /etc/ssl/certs/revoke.crl
    #tls_fingerprint [fingerprint]
    #tls_key_file [file]
    #tls_cert_file [file]
    tls_certcheck on
    #tls_priorities [priorities]
    #dsn_notify (off|condition)
    #dsn_return (off|amount)
    #domain argument
    #keepbcc off
    logfile /var/log/mail.log
    syslog on
    account default : gmail
    EOF
    chmod 0400 /root/.msmtprc
    
       ## In testing .. auto command
    # echo -e "1\n4096\n\ny\n$MYUSRMAIL\n$MYEMAIL\nmy key\nO\n$PWDMAIL\n$PWDMAIL\n" | gpg --full-generate-key 
    ##
    gpg --full-generate-key
    gpg --output revoke.asc --gen-revoke $MYEMAIL
    echo -e "$PWDEMAIL\n" | gpg -e -o /root/msmtp-mail.gpg --recipient $MYEMAIL
    echo "export GPG_TTY=\$(tty)" >> .baschrc	
    chmod 400 msmtp-mail.gpg
    
    echo "Hello there" | msmtp --debug $MYEMAIL
    echo"######################
    ## MSMTP Configured ##
    ######################"
    ```
    
完成！！;)
([目录](#table-of-contents))

<a id="gmail-and-exim4-as-mta-with-implicit-tls"></a>
### 使用隐式 TLS 将 Gmail 和 Exim4 配置为 MTA

#### 原因

除非你打算搭建自己的邮件服务器，否则需要一种从服务器发送电子邮件的方法。这对于系统警报和消息非常重要。

可以使用任意 Gmail 账户。我建议专门为此服务器创建一个账户。这样，即使服务器**确实**遭到入侵，恶意行为者也不会获得你主账户的任何密码。当然，如果已启用 2FA/MFA 并使用应用密码，仅凭应用密码，恶意行为者能做的事情并不多，但为什么要冒险？

网上有许多指南介绍如何使用 STARTTLS 将 Gmail 配置为 MTA，其中包括[本指南的先前版本](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/tree/cc5edcae1cf846dd250e76b121e721d836481d2f#configure-gmail-as-mta)。使用 STARTTLS 时，会先建立一个**未加密**的初始连接，然后再升级为加密的 TLS 或 SSL 连接。下面介绍的方法则会从一开始就建立加密的 TLS 连接。

此外，正如 [issue #29](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/issues/29) 和[此处](https://blog.dhampir.no/content/exim4-line-length-in-debian-stretch-mail-delivery-failed-returning-message-to-sender)所述，exim4 在处理含长行的消息时会失败。本节也将修复此问题。

** **重要** ** 如 [#106](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/issues/106) 所述，Google 已不再允许使用账户密码进行身份验证。必须启用 2FA，然后使用应用密码。

#### 目标

- 配置 `mail`，以便使用 [Gmail](https://mail.google.com/) 从服务器发送电子邮件
- 为 exim4 提供长行支持

#### 参考资料

- 感谢 [remyabel](https://github.com/remyabel) 找到了使用 TLS 实现此功能的方法，具体记录在 [issue #24](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/issues/24) 和 [pull request #26](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/pull/26) 中。
- https://wiki.debian.org/Exim
- https://wiki.debian.org/GmailAndExim4
- https://www.exim.org/exim-html-current/doc/html/spec_html/ch-encrypted_smtp_connections_using_tlsssl.html
- https://php.quicoto.com/setup-exim4-to-use-gmail-in-ubuntu/
- https://www.fastmail.com/help/technical/ssltlsstarttls.html
- exim4 处理长行消息时会失败——[issue #29](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/issues/29) 和 https://blog.dhampir.no/content/exim4-line-length-in-debian-stretch-mail-delivery-failed-returning-message-to-sender
- https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/issues/106

#### 操作步骤

1. 安装 exim4。还需要安装 openssl 和 ca-certificates。

    在基于 Debian 的系统上：

    ``` bash
    sudo apt install exim4 openssl ca-certificates
    ```

1. 配置 exim4：

    对于基于 Debian 的系统：
    ``` bash
    sudo dpkg-reconfigure exim4-config
    ```

    系统会提示回答一些问题：

    |提示|答案|
    |--:|--|
    |邮件配置的常规类型|`mail sent by smarthost; no local mail`|
    |系统邮件名称|`localhost`|
    |用于侦听传入 SMTP 连接的 IP 地址|`127.0.0.1; ::1`|
    |接受邮件的其他目标地址|（默认值）|
    |本地用户的可见域名|`localhost`|
    |传出 smarthost 的 IP 地址或主机名|`smtp.gmail.com::465`|
    |尽量减少 DNS 查询次数（按需拨号）？|`No`|
    |是否将配置拆分为小文件？|`No`|

1. 备份 `/etc/exim4/passwd.client`：

    ``` bash
    sudo cp --archive /etc/exim4/passwd.client /etc/exim4/passwd.client-COPY-$(date +"%Y%m%d%H%M%S")
    ```

1. 将类似下面的行添加到 `/etc/exim4/passwd.client`

    ```
    smtp.gmail.com:yourAccount@gmail.com:yourPassword
    *.google.com:yourAccount@gmail.com:yourPassword
    ```

    **注意事项**：
    - 将 `yourAccount@gmail.com` 和 `yourPassword` 替换为你的详细信息。如果 Gmail 已启用 2FA/MFA，则需要在此处创建并使用应用密码。
    - 始终检查 `host smtp.gmail.com`，以获得应列出的最新域名。

1. 此文件包含 Gmail 密码，因此需要限制其访问权限：

    ``` bash
    sudo chown root:Debian-exim /etc/exim4/passwd.client
    sudo chmod 640 /etc/exim4/passwd.client
    ```

1. 下一步是创建一个 TLS 证书，exim4 将使用它与 `smtp.gmail.com` 建立加密连接。可以使用自己的证书，例如来自 [Let's Encrypt](https://letsencrypt.org/) 的证书，也可以使用 openssl 自行创建。这里将使用 exim4 附带的脚本，该脚本会调用 openssl 创建证书：

    ``` bash
    sudo bash /usr/share/doc/exim4-base/examples/exim-gencert
    ```

    > ```
    > [*] Creating a self signed SSL certificate for Exim!
    >     This may be sufficient to establish encrypted connections but for
    >     secure identification you need to buy a real certificate!
    > 
    >     Please enter the hostname of your MTA at the Common Name (CN) prompt!
    > 
    > Generating a RSA private key
    > ..........................................+++++
    > ................................................+++++
    > writing new private key to '/etc/exim4/exim.key'
    > -----
    > You are about to be asked to enter information that will be incorporated
    > into your certificate request.
    > What you are about to enter is what is called a Distinguished Name or a DN.
    > There are quite a few fields but you can leave some blank
    > For some fields there will be a default value,
    > If you enter '.', the field will be left blank.
    > -----
    > Country Code (2 letters) [US]:[redacted]
    > State or Province Name (full name) []:[redacted]
    > Locality Name (eg, city) []:[redacted]
    > Organization Name (eg, company; recommended) []:[redacted]
    > Organizational Unit Name (eg, section) []:[redacted]
    > Server name (eg. ssl.domain.tld; required!!!) []:localhost
    > Email Address []:[redacted]
    > [*] Done generating self signed certificates for exim!
    >     Refer to the documentation and example configuration files
    >     over at /usr/share/doc/exim4-base/ for an idea on how to enable TLS
    >     support in your mail transfer agent.
    > ```

1. 创建文件 `/etc/exim4/exim4.conf.localmacros` 并添加以下内容，让 exim4 使用 TLS 和端口 465，并[修复 exim4 的长行问题](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/issues/29)：

    ```
    MAIN_TLS_ENABLE = 1
    REMOTE_SMTP_SMARTHOST_HOSTS_REQUIRE_TLS = *
    TLS_ON_CONNECT_PORTS = 465
    REQUIRE_PROTOCOL = smtps
    IGNORE_SMTP_LINE_LENGTH_LIMIT = true
    ```

    [懒人版](#editing-configuration-files---for-the-lazy)：

    ``` bash
    cat << EOF | sudo tee /etc/exim4/exim4.conf.localmacros
    MAIN_TLS_ENABLE = 1
    REMOTE_SMTP_SMARTHOST_HOSTS_REQUIRE_TLS = *
    TLS_ON_CONNECT_PORTS = 465
    REQUIRE_PROTOCOL = smtps
    IGNORE_SMTP_LINE_LENGTH_LIMIT = true
    EOF
    ```

1. 备份 exim4 配置文件 `/etc/exim4/exim4.conf.template`：

    ``` bash
    sudo cp --archive /etc/exim4/exim4.conf.template /etc/exim4/exim4.conf.template-COPY-$(date +"%Y%m%d%H%M%S")
    ```

1. 在 `/etc/exim4/exim4.conf.template` 中的 `.ifdef REMOTE_SMTP_SMARTHOST_HOSTS_REQUIRE_TLS ... .endif` 块之后添加以下内容：

    ```
    .ifdef REQUIRE_PROTOCOL
      protocol = REQUIRE_PROTOCOL
    .endif
    ```

    > ```
    > .ifdef REMOTE_SMTP_SMARTHOST_HOSTS_REQUIRE_TLS
    >   hosts_require_tls = REMOTE_SMTP_SMARTHOST_HOSTS_REQUIRE_TLS
    > .endif
    > .ifdef REQUIRE_PROTOCOL
    >     protocol = REQUIRE_PROTOCOL
    > .endif
    > .ifdef REMOTE_SMTP_HEADERS_REWRITE
    >   headers_rewrite = REMOTE_SMTP_HEADERS_REWRITE
    > .endif
    > ```

    [懒人版](#editing-configuration-files---for-the-lazy)：

    ``` bash
    sudo sed -i -r -e '/^.ifdef REMOTE_SMTP_SMARTHOST_HOSTS_REQUIRE_TLS$/I { :a; n; /^.endif$/!ba; a\# added by '"$(whoami) on $(date +"%Y-%m-%d @ %H:%M:%S")"'\n.ifdef REQUIRE_PROTOCOL\n    protocol = REQUIRE_PROTOCOL\n.endif\n# end add' -e '}' /etc/exim4/exim4.conf.template
    ```

1. 在 `/etc/exim4/exim4.conf.template` 的 `.ifdef MAIN_TLS_ENABLE` 块内添加以下内容：

    ```
    .ifdef TLS_ON_CONNECT_PORTS
      tls_on_connect_ports = TLS_ON_CONNECT_PORTS
    .endif
    ```

    > ```
    > .ifdef MAIN_TLS_ENABLE
    > .ifdef TLS_ON_CONNECT_PORTS
    >     tls_on_connect_ports = TLS_ON_CONNECT_PORTS
    > .endif
    > ```

    [懒人版](#editing-configuration-files---for-the-lazy)：

    ``` bash
    sudo sed -i -r -e "/\.ifdef MAIN_TLS_ENABLE/ a # added by $(whoami) on $(date +"%Y-%m-%d @ %H:%M:%S")\n.ifdef TLS_ON_CONNECT_PORTS\n    tls_on_connect_ports = TLS_ON_CONNECT_PORTS\n.endif\n# end add" /etc/exim4/exim4.conf.template
    ```
    
1. 更新 exim4 配置以使用 TLS，然后重启服务：

    ``` bash
    sudo update-exim4.conf
    sudo service exim4 restart
    ```

1. 如果正在使用 [UFW](#firewall-with-ufw-uncomplicated-firewall)，则需要允许端口 465 上的出站流量。为此，我们将创建一个自定义 UFW 应用程序配置文件，然后启用它。创建文件 `/etc/ufw/applications.d/smtptls`，添加以下内容，然后运行 `ufw allow out smtptls comment 'open TLS port 465 for use with SMPT to send e-mails'`：

    ```
    [SMTPTLS]
    title=SMTP through TLS
    description=This opens up the TLS port 465 for use with SMPT to send e-mails.
    ports=465/tcp
    ```

    [懒人版](#editing-configuration-files---for-the-lazy)：

    ``` bash
    cat << EOF | sudo tee /etc/ufw/applications.d/smtptls
    [SMTPTLS]
    title=SMTP through TLS
    description=This opens up the TLS port 465 for use with SMPT to send e-mails.
    ports=465/tcp
    EOF

    sudo ufw allow out smtptls comment 'open TLS port 465 for use with SMPT to send e-mails'
    ```

1. 将类似以下内容的行添加到 `/etc/aliases`，以添加一些邮件别名，这样就可以向本地账户发送电子邮件：

    ```
    user1: user1@gmail.com
    user2: user2@gmail.com
    ...
    ```

    需要添加服务器上存在的所有本地账户。

1. 测试设置：

    ```
    echo "test" | mail -s "Test" email@gmail.com
    sudo tail /var/log/exim4/mainlog
    ```

([目录](#table-of-contents))

<a id="separate-iptables-log-file"></a>
### 单独的 iptables 日志文件

#### 原因

总有一天，你会需要查看 iptables 日志。让所有 iptables 日志都写入自己的文件，会让查找所需内容容易得多。

#### 参考资料

- https://blog.shadypixel.com/log-iptables-messages-to-a-separate-file-with-rsyslog/
- https://gist.github.com/netson/c45b2dc4e835761fbccc
- https://www.rsyslog.com/doc/v8-stable/configuration/actions.html

#### 操作步骤

1. 第一步是让防火墙在所有日志条目前添加某个唯一字符串作为前缀。如果直接使用 iptables，可以为所有规则添加类似 `--log-prefix "[IPTABLES] "` 的内容。我们已在[安装 psad 的第 4 步](#psad_step4)中完成此操作。

1. 为防火墙日志添加前缀后，需要让 rsyslog 将这些行发送到单独的文件。为此，请创建文件 `/etc/rsyslog.d/10-iptables.conf` 并添加以下内容：

    ```
    :msg, contains, "[IPTABLES] " /var/log/iptables.log
    & stop
    ```
    
    如果预计防火墙会记录大量数据，请在文件名前添加 `-`，[“以避免每次记录日志后同步文件”](https://www.rsyslog.com/doc/v8-stable/configuration/actions.html#regular-file)。例如：

    ```
    :msg, contains, "[IPTABLES] " -/var/log/iptables.log
    & stop
    ```

    **注意**：请记得将前缀更改为你所使用的值。
    
    [懒人版](#editing-configuration-files---for-the-lazy)：

    ``` bash
    cat << EOF | sudo tee /etc/rsyslog.d/10-iptables.conf
    :msg, contains, "[IPTABLES] " /var/log/iptables.log
    & stop
    EOF
    ```

1. 由于现在将防火墙消息记录到另一个文件中，因此需要告诉 psad 新文件的位置。编辑 `/etc/psad/psad.conf`，将 `IPT_SYSLOG_FILE` 设置为日志文件的路径。例如：

    ```
    IPT_SYSLOG_FILE /var/log/iptables.log;
    ```
    
    **注意**：请记得将前缀更改为你所使用的值。
    
    [懒人版](#editing-configuration-files---for-the-lazy)：
    
    ``` bash
    sudo sed -i -r -e "s/^(IPT_SYSLOG_FILE\s+)([^;]+)(;)$/# \1\2\3       # commented by $(whoami) on $(date +"%Y-%m-%d @ %H:%M:%S")\n\1\/var\/log\/iptables.log\3       # added by $(whoami) on $(date +"%Y-%m-%d @ %H:%M:%S")/" /etc/psad/psad.conf 
    ```

1. 重启 psad 和 rsyslog 以激活更改（也可以重启系统）：

    ``` bash
    sudo psad -R
    sudo psad --sig-update
    sudo psad -H
    sudo service rsyslog restart
    ```

1. 最后，需要让 logrotate 轮转新的日志文件，以免文件变得过大并占满磁盘。创建文件 `/etc/logrotate.d/iptables` 并添加以下内容：

    ```
    /var/log/iptables.log
    {
        rotate 7
        daily
        missingok
        notifempty
        delaycompress
        compress
        postrotate
            invoke-rc.d rsyslog rotate > /dev/null
        endscript
    }
    ```
    
    [懒人版](#editing-configuration-files---for-the-lazy)：
    
    ``` bash
    cat << EOF | sudo tee /etc/logrotate.d/iptables
    /var/log/iptables.log
    {
        rotate 7
        daily
        missingok
        notifempty
        delaycompress
        compress
        postrotate
            invoke-rc.d rsyslog rotate > /dev/null
        endscript
    }
    EOF
    ```

([目录](#table-of-contents))

<a id="left-over"></a>
## 其他信息

<a id="contacting-me"></a>
### 联系我

如有任何问题、评论、疑虑、反馈或故障，请提交[新建 issue](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server/issues/new)。

([目录](#table-of-contents))

<a id="helpful-links"></a>
### 实用链接

- [https://github.com/pratiktri/server_init_harden](https://github.com/pratiktri/server_init_harden)——一个 Bash 脚本，可自动执行一些需要在新 Linux 服务器上完成的任务，为其提供基本安全保护。

([目录](#table-of-contents))

<a id="acknowledgments"></a>
### 致谢

- https://www.reddit.com/r/linuxquestions/comments/aopzl7/new_guide_created_by_me_how_to_secure_a_linux/
- https://www.reddit.com/r/selfhosted/comments/aoxd4l/new_guide_created_by_me_how_to_secure_a_linux/
- https://news.ycombinator.com/item?id=19177435#19178618
- https://www.reddit.com/r/linuxadmin/comments/arx7xo/howtosecurealinuxserver_an_evolving_howto_guide/
- https://www.reddit.com/r/linux/comments/arx7st/howtosecurealinuxserver_an_evolving_howto_guide/
- https://github.com/moltenbit/How-To-Secure-A-Linux-Server-With-Ansible

([目录](#table-of-contents))

<a id="license"></a>
<a id="license-and-copyright"></a>
### 许可证与版权

[![CC-BY-SA](https://i.creativecommons.org/l/by-sa/4.0/88x31.png)](http://creativecommons.org/licenses/by-sa/4.0/)

[How To Secure A Linux Server](https://github.com/imthenachoman/How-To-Secure-A-Linux-Server) 由 [Anchal Nigam](https://github.com/imthenachoman) 创作，采用 [署名—相同方式共享 4.0 协议国际版](http://creativecommons.org/licenses/by-sa/4.0) 许可。

完整许可证见 [LICENSE](LICENSE.txt)。

([目录](#table-of-contents))
