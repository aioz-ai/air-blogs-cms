---
last_modified_on: "2021-09-20"
title: Smart Transcoding (part 1) Introduction.
description: A series of Smart Transcoding based on AI.
series_position: 10
author_github: https://github.com/Gazeal
tags: ["type: tutorial", "level: advance"]
---


The growth of the video streaming business has led in an increase in demand for transcoding services from a wide range of consumers. Recent advancements in blockchain technology enable certain businesses to implement decentralized collaborative transcoding using device-to-device (D2D) networks, which pick a set of transcoders to execute transcoding collaboratively. In order to offer efficient and trustworthy transcoding services for block chain-enabled D2D transcoding systems, it is critical to collaboratively develop transcoder selection, job scheduling, and resource allocation algorithms.

![Fig-0](https://vision.aioz.io/f/d95290a29cda419bb702/?dl=1)

In this series, I want to introduce how AI-powered agents can burst the process of video transcoding task. Firstly, we need to investigate the Blockchain-enabled transcoding systems to understand the importance of transcoding in practice.
## Cloud-based video transcoding for dealing with computation intensive and time-consuming

With the exponential rise of mobile devices, video streaming services and applications have grown immensely in popularity and consumer interest. Online videos are exploding in popularity, accounting approximately 80 percent of all consumer Internet traffic by 2019 [1]. To support a wide range of user devices (smartphones, laptop and desktop computers, TVs, and so on), the source video streams must be converted into multiple representations (bitrates, formats, resolutions, codecs, and so on) with various quality of service (QoS) types, which is extremely computationally intensive and time-consuming [2]. To address this issue, cloud-based video transcoding is seen as a viable solution and has been extensively used by well-known firms such as Netflix, Amazon Prime, Vimeo, and Youtube, among others. [3]–[5].

![Fig-1](https://www.researchgate.net/profile/Adnan-Ashraf-10/publication/267052501/figure/fig6/AS:613856846032899@1523366285180/System-architecture-of-the-cloud-based-on-demand-video-transcoding-service.png)
*<center>**Figure 1**: Cloud-based video transcoding system ([Source](https://www.google.com/url?sa=i&url=https%3A%2F%2Fwww.researchgate.net%2Ffigure%2FSystem-architecture-of-the-cloud-based-on-demand-video-transcoding-service_fig6_267052501&psig=AOvVaw2acMo1RwWbrZ44Gf4oS1dw&ust=1631760705206000&source=images&cd=vfe&ved=0CAsQjRxqFwoTCJDH5rv8__ICFQAAAAAdAAAAABAI)).</center>*

![Fig-2](https://slickdeals.net/blog/wp-content/uploads/2019/04/streaming-services-hero-1.png)
*<center>**Figure 2**: Video streaming services that use Cloud-based video transcoding  system ([Source](https://www.google.com/url?sa=i&url=https%3A%2F%2Fslickdeals.net%2Farticle%2Flist%2Fbest-free-trials-popular-online-video-streaming-services%2F&psig=AOvVaw0roFrdRtXrZl6achN0QYDA&ust=1631760654690000&source=images&cd=vfe&ved=0CAsQjRxqFwoTCIi3vqf9__ICFQAAAAAdAAAAABAJ)).</center>*

Despite their ability to deliver high-quality transcoding services, today's cloud-based transcoding platforms face certain challenges: 1) A long round-trip delay owing to the remote cloud server. 2) A significant strain on the backhaul lines while uploading and retrieving video material. 3) Transcoding services provide a low level of transparency and security.

## Blockchain-Enabled video transcoding

To address the drawbacks of centralized cloud-based transcoding systems, a number of firms (such as Transcodium [6], for example) are leveraging developing blockchain technology to enable “crowdsourcing” transcoding with flexible monetization methods. Meanwhile, with end-device storage and processing capability increasing, blockchain may be deployed via device-to-device (D2D) networks, which provide a decentralized platform [7–11].

![Fig-3](https://d3i71xaburhd42.cloudfront.net/e87768672ee1ccc4d31d02fe158b68d9b6a5bd0b/3-Figure1-1.png)
*<center>**Figure 3**: The architecture of video transcoding and delivery with blockchain and mobile edge computing. ([Source](https://www.google.com/url?sa=i&url=https%3A%2F%2Fwww.semanticscholar.org%2Fpaper%2FVideo-Transcoding-and-Delivery-with-Blockchain-and-Liu-Li%2Fe87768672ee1ccc4d31d02fe158b68d9b6a5bd0b&psig=AOvVaw3-12T5ro7HCjqNOW3lSPNP&ust=1631763295276000&source=images&cd=vfe&ved=0CAsQjRxqFwoTCIDf7o-GgPMCFQAAAAAdAAAAABAN)).</center>*

In comparison to existing cloud-based transcoding platforms, these upcoming blockchain-enabled ones can deliver more efficient transcoding services by dividing big video files into little bits and allocating them to a group of transcoders without the involvement of any intermediaries. Furthermore, content providers establish direct partnerships with transcoders via D2D networks, which can alleviate the high strain of backhaul cables. Furthermore, the encrypted smart contract can improve the security and transparency of transcoding jobs (e.g., file size, file length, task payment) [9], [12].

## Current remaining problems of Blockchain-Enabled video transcoding

Nonetheless, blockchain-enabled transcoding over D2D networks is still in its early stages. Although it can stimulate distributed transcoder collaboration through incentive mechanisms, the transcoding efficiency and trustworthiness of blockchain-enabled collaborative transcoding have not been well addressed. There is, for example, a lack of an integrated assessment process for transcoder selection. Most existing blockchain-enabled services select transcoders at random or solely based on stakes (e.g., Theta [13], Livepeer [14]). However, the computing and communication capabilities of each transcoder have a significant influence on transcoding efficiency and should be taken into account as well. Meanwhile, to increase transcoding trustworthiness, the reputation method should be used for transcoder selection. Furthermore, transcoding job scheduling and the allocation of communication and computing resources have an impact on transcoding efficiency. As a result, how to jointly construct the transcoding job scheduling and resource allocation system is equally essential.

As a result, it is critical to provide an efficient system for addressing the concerns of transcoder selection, work scheduling, and resource allocation. Nonetheless, the dynamic and complicated properties of transcoding systems (e.g., workloads, QoS requirements, and candidate attributes) make standard optimization approaches challenging to use.

## Next section introduction
In the next section, we will investigate how to deal with the Blockchain-Enabled video transcoding using deep reinforcement learning, an AI-powered optimization method.

## References
[1] Cisco, “Cisco visual networking index: Global mobile data traffic forecast update, 2016-2021 white paper,” Feb. 2017.

[2] G. Gao, Y. Wen, and J. Cai, “vcache: Supporting cost-efficient adaptive bitrate streaming via nfv-based virtual caching,” IEEE MultiMedia, pp.1–1, June 2017.

[3] Z. Huang, C. Mei, L. Li, and T. Woo, “Cloudstream: Delivering highquality streaming videos through a cloud-based svc proxy,” in IEEE INFOCOM Mini Conf. Shanghai, Apr. 2011, pp. 201–205.

[4] M. Kim, Y. Cui, S. Han, and H. Lee, “Towards efficient design and implementation of a hadoop-based distributed video transcoding system in cloud computing environment,” International Journal of Multimedia and Ubiquitous Engineering, vol. 8, no. 2, pp. 213–224, Mar. 2013.

[5] Q. He, C. Zhang, X. Ma, and J. Liu, “Fog-based transcoding for crowdsourced video livecast,” IEEE Comm. Mag., vol. 55, no. 4, pp.28–33, April 2017.

[6] Transcodium, “Transcodium: A decentralized peer-to-peer media editing, transcoding & distribution platform,” 2017.

[7] C. Wright and A. Serguieva, “Blockchain-based payment collection supervision system using pervasive bitcoin digital wallet,” in Proc. IEEE Int’l Conf. on Big Data. Boston, MA, Dec. 2017, pp. 4255–4264.

[8] D. Magazzeni, P. McBurney, and W. Nash, “Validation and verification of smart contracts: A research agenda,” Computer, vol. 50, no. 9, pp.50–57, Spet. 2017.

[9] F. R. Yu, J. M. Liu, Y. He, P. B. Si, and Y. H. Zhang, “Virtualization for distributed ledger technology (vdlt),” IEEE Access, vol. 6, pp. 25 019–25 028, April 2018.

[10] V. Gatteschi, F. Lamberti, C. Demartini, C. Pranteda, and V. Santamar?a, “To blockchain or not to blockchain: That is the question,” IEEE Computer Society, Mar./Apr. 2018.

[11] M. Liu, F. R. Yu, Y. Teng, V. C. M. Leung, and M. Song, “Joint computation offloading and content caching for wireless blockchain networks,” in Proc. IEEE Conf. on Computer Commun. Workshops (INFOCOM WKSHPS). Honolulu, HI, April 2018, pp. 1–6.

[12] A. Kosba, A. Miller, E. Shi, Z. Wen, and C. Papamanthou, “Hawk: The blockchain model of cryptography and privacy-preserving smart
contracts,” in Proc. IEEE Symposium on Security and Privacy (SP). San Jose, CA, May. 2016, pp. 839–858.

[13] SLIVER.tv, “Theta: A decentralized video streaming network, powered by a new blockchain and token v1.6,” Jan. 2018.

[14] D. Petkanic and E. Tang, “Livepeer whitepaper: Protocol and economic incentives for a decentralized live video streaming network,” Dec. 2017.
