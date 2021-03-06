# CrackSSH!
*調査対象のサーバーで使われている公開鍵を入手した。噂によると、この鍵には問題があるらしいが...。なんとかして侵入しなければ。*
*Target: frt.hongo.wide.ad.jp*
*Port: 30022*

---


题目中给了一个公钥:

```asciiarmor
ssh-rsa AAAAB3NzaC1yc2EAAACBAWKA1hYjuvhxiwCGKsG+nbLj/iYy6pRwkkka64J6L+VLPp4K3JVSREEzmztAWxjkhGOleol3vzDRqR2J+4nSVOI9FhJyiBdSgECmXJYojGVSU56bCMdcysEkKYVz5e0+xQAjZDrotpm+FT0VAdwdWuZM68zZY8DE9H2uo9daHCf/AAAAgQIB+Y+6jm9xvNibnZLIoAvIVv1GflbjQ5AoKp52yPq+3nRr1N1aalXhHV1pXcwa1yra81+DFDsu4bdpPC7f25pLriBZKaSNT7K0+sRQdP50iBaYjsF2Cyg8HjoeGaXVkh3bOw2V2WwUsU4qEr9TjPbMzrCCxkFDQPnwOwmiWQM8GQ== tsukushi@frt.hongo.wide.ad.jp
```

参考知乎([ssh-keygen生成的id_rsa文件的格式](https://zhuanlan.zhihu.com/p/33949377))上的的一篇文章:

首先,`awk '{print $2}' crackssh.pub | base64 -d | hexdump -C`,得到:


```assembly
00000000  00 00 00 07 73 73 68 2d  72 73 61 00 00 00 81 01  |....ssh-rsa.....|
00000010  62 80 d6 16 23 ba f8 71  8b 00 86 2a c1 be 9d b2  |b...#..q...*....|
00000020  e3 fe 26 32 ea 94 70 92  49 1a eb 82 7a 2f e5 4b  |..&2..p.I...z/.K|
00000030  3e 9e 0a dc 95 52 44 41  33 9b 3b 40 5b 18 e4 84  |>....RDA3.;@[...|
00000040  63 a5 7a 89 77 bf 30 d1  a9 1d 89 fb 89 d2 54 e2  |c.z.w.0.......T.|
00000050  3d 16 12 72 88 17 52 80  40 a6 5c 96 28 8c 65 52  |=..r..R.@.\.(.eR|
00000060  53 9e 9b 08 c7 5c ca c1  24 29 85 73 e5 ed 3e c5  |S....\..$).s..>.|
00000070  00 23 64 3a e8 b6 99 be  15 3d 15 01 dc 1d 5a e6  |.#d:.....=....Z.|
00000080  4c eb cc d9 63 c0 c4 f4  7d ae a3 d7 5a 1c 27 ff  |L...c...}...Z.'.|
00000090  00 00 00 81 02 01 f9 8f  ba 8e 6f 71 bc d8 9b 9d  |..........oq....|
000000a0  92 c8 a0 0b c8 56 fd 46  7e 56 e3 43 90 28 2a 9e  |.....V.F~V.C.(*.|
000000b0  76 c8 fa be de 74 6b d4  dd 5a 6a 55 e1 1d 5d 69  |v....tk..ZjU..]i|
000000c0  5d cc 1a d7 2a da f3 5f  83 14 3b 2e e1 b7 69 3c  |]...*.._..;...i<|
000000d0  2e df db 9a 4b ae 20 59  29 a4 8d 4f b2 b4 fa c4  |....K. Y)..O....|
000000e0  50 74 fe 74 88 16 98 8e  c1 76 0b 28 3c 1e 3a 1e  |Pt.t.....v.(<.:.|
000000f0  19 a5 d5 92 1d db 3b 0d  95 d9 6c 14 b1 4e 2a 12  |......;...l..N*.|
00000100  bf 53 8c f6 cc ce b0 82  c6 41 43 40 f9 f0 3b 09  |.S.......AC@..;.|
00000110  a2 59 03 3c 19                                    |.Y.<.|
00000115
```

* 前4个字节(00 00 00 07)表示接下来的数据块是7个字节,接下来7个字节的内容是73 73 68 2d 72 73 61,正好是`ssh-rsa`的ASCII码

* 接下来四个字节(00 00 00 81)表示接下来的数据块是0x81个字节,得到e的16进制

* 再接下来四个字节(00 00 00 81)表示接下来的数据块是0x81个字节,得到n的16进制

  ```shell
  ~$ echo "$((16#00000081))"
  129
  ~$ N=$(awk '{print $2}' ~/.ssh/id_rsa.pub | base64 -d | hexdump -ve '1/1 "%.2x"')
  ~$ e=${N: 30:129*2} && echo $e
  0100010000020100e5f95a4428736b17d113d90b86eba7d9052ebec087c808fd3e5704a10b2df04638b309f0e7cae4b0cd5bcefbfae5c28f681edbedaf10e35c77201380f4d309b337c8e0c62e815a967d18c9a4642fb1ebc44ea3a4a75335d097135895a7604e1662c6df43d61212f389288f4e717e4e6ebec06ce1fc5b1d4c2c
  ~$ n=${N: -129*2} && echo $n
  bf582bcc145ea7b5b398b0c51c06be2d367248f6c247653e928bb317a290d7e7a99c169999cf0dbc56f875948d236da894abf2c8b4aac54d5dcb2b6ac8bfb1f9d87416a94a46e70d3083811a0397dcc8abea0b7e1b88fd230f05fd5d45de5bdee21d535aea684749a5c576d8b6e96aa358175f8f7666f27e9729fab06e25ac8021
  ```

> 官方的Writeup给了另一种做法,可以直接得到e和n的16进制表示形式
> ```shell
> ~$ ssh-keygen -f crackssh.pub -e -m pem | openssl asn1parse
>     0:d=0  hl=4 l= 264 cons: SEQUENCE          
>     4:d=1  hl=3 l= 129 prim: INTEGER  :0201F98FBA8E6F71BCD89B9D92C8A00BC856FD467E56E34390282A9E76C8FABEDE746BD4DD5A6A55E11D5D695DCC1AD72ADAF35F83143B2EE1B7693C2EDFDB9A4BAE205929A48D4FB2B4FAC45074FE748816988EC1760B283C1E3A1E19A5D5921DDB3B0D95D96C14B14E2A12BF538CF6CCCEB082C6414340F9F03B09A259033C19
>   136:d=1  hl=3 l= 129 prim: INTEGER  :016280D61623BAF8718B00862AC1BE9DB2E3FE2632EA947092491AEB827A2FE54B3E9E0ADC95524441339B3B405B18E48463A57A8977BF30D1A91D89FB89D254E23D1612728817528040A65C96288C6552539E9B08C75CCAC124298573E5ED3EC50023643AE8B699BE153D1501DC1D5AE64CEBCCD963C0C4F47DAEA3D75A1C27FF
> ```
>

使用[RsaCtfTool](https://github.com/Ganapati/RsaCtfTool)生成私钥,保存至`id_rsa`

```shell
python RsaCtfTool.py -n 360925413365609656207284763303112593050686426607629131354843699618905677197872793512380288223361149508460688151102823348462592916817609977273908821217493993702786929282477487755465976082059834867631026295714550319202482180891845062064382568022072228888091051431136923983143306662931216184662445381040847666201 -e 248940659700671391171916045022225211367167934215525303038734152650593067612113589541083076628705613883775652505492831370527586438096113903892713520850387855997035509546247913887222055672708066391999421835495881798128330308530099218984443115901043292942963247939575084326452874538239309850357410618060448737279 --private
```

```asciiarmor
-----BEGIN RSA PRIVATE KEY-----
MIICOQIBAAKBgQIB+Y+6jm9xvNibnZLIoAvIVv1GflbjQ5AoKp52yPq+3nRr1N1a
alXhHV1pXcwa1yra81+DFDsu4bdpPC7f25pLriBZKaSNT7K0+sRQdP50iBaYjsF2
Cyg8HjoeGaXVkh3bOw2V2WwUsU4qEr9TjPbMzrCCxkFDQPnwOwmiWQM8GQKBgQFi
gNYWI7r4cYsAhirBvp2y4/4mMuqUcJJJGuuCei/lSz6eCtyVUkRBM5s7QFsY5IRj
pXqJd78w0akdifuJ0lTiPRYScogXUoBAplyWKIxlUlOemwjHXMrBJCmFc+XtPsUA
I2Q66LaZvhU9FQHcHVrmTOvM2WPAxPR9rqPXWhwn/wIgNHyza85w/fnNPOZwpCTM
OZ6242GZZRcNX4iFJaXeun8CQQEL5ReRSsEcF106aHY8Yohd4FxaChPms4M8/DXP
+WMC6GT0qWRj3LZYkndvP2WxVOoZYxf0rQc+ew9rFVaZ6OotAkEB6ydXjs82dKS/
KS3ffoUyj4oh7viR9j3fH5WBep9S0MeyVZc16Cwj0mPZDRbL1n7Cs0oSZD/A4FzO
OgWAxc2pHQIgNHyza85w/fnNPOZwpCTMOZ6242GZZRcNX4iFJaXeun8CIDR8s2vO
cP35zTzmcKQkzDmetuNhmWUXDV+IhSWl3rp/AkBdum7eZMxE/VYX7QV9xND/bpn2
/MHD4BHF8c/MirawY5HC/RTviwnfpXAoF4ArBY1ZG3nCn9L19v/sUcFQKC9X
-----END RSA PRIVATE KEY-----
```

```shell
chmod 600 id_rsa
ssh tsukushi@frt.hongo.wide.ad.jp -p 30022 -i id_rsa
```

**flag:**`TsukuCTF{D0nt_use_w34k_RS4_key_generat10n}`

