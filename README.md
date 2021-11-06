tixoncash
TXC
tixoncash
跳到内容
泰克森币
/
泰克森现金
民众
代码
问题
拉取请求
行动
项目
维基
安全
洞察力
泰克森现金/源文件/ pubkey.cpp
@tixoncoin
tixoncoin 源上传
 1 位 贡献者
可执行文件  140 行 (129 sloc)  3.83 KB
//版权所有 (c) 2009-2014 比特币开发者
//在 MIT 软件许可下分发，见随附
//复制文件或 http://www.opensource.org/licenses/mit-license.php。

#包含 “ pubkey.h ”

#包含 “ eccryptoverify.h ”

# ifdef USE_SECP256K1
#包含 < secp256k1.h >
#其他
#包含 “ ecwrapper.h ”
＃ENDIF

bool  CPubKey::Verify ( const uint256& hash, const std::vector< unsigned  char >& vchSig) const
{
    如果(! IsValid ())
        返回 假；
# ifdef USE_SECP256K1
    如果（secp256k1_ecdsa_verify（（const的 无符号 字符*）＆散列，32，＆vchSig [ 0 ]，vchSig。大小（），开始（），大小（））！= 1）
        返回 假；
#其他
    CECKey 密钥；
    if (! key.SetPubKey ( begin (), size ()))
        返回 假；
    如果（！键确认（哈希，vchSig））
        返回 假；
＃ENDIF
    返回 真；
}

bool  CPubKey::RecoverCompact ( const uint256& hash, const std::vector< unsigned  char >& vchSig)
{
    如果（vchSig。大小（）！= 65）
        返回 假；
    int recid = (vchSig[ 0 ] - 27 ) & 3 ;
    bool  fComp = ((vchSig[ 0 ] - 27 ) & 4 ) != 0 ;
# ifdef USE_SECP256K1
    int pubkeylen = 65；
    if (! secp256k1_ecdsa_recover_compact (( const  unsigned  char *)&hash, 32 , &vchSig[ 1 ], ( unsigned  char *) begin (), &pubkeylen, fComp , recid))
        返回 假；
    断言(( int ) size () == pubkeylen);
#其他
    CECKey 密钥；
    如果（！键。恢复（散列，＆vchSig [ 1 ]，recid））
        返回 假；
    std::vector<无符号 字符>公钥；
    钥匙。GetPubKey（公钥，fComp）；
    设置（PUBKEY。开始（），PUBKEY。结束（））;
＃ENDIF
    返回 真；
}

bool  CPubKey::IsFullyValid () const
{
    如果(! IsValid ())
        返回 假；
# ifdef USE_SECP256K1
    如果（！secp256k1_ecdsa_pubkey_verify（开始（），大小（）））
        返回 假；
#其他
    CECKey 密钥；
    if (! key.SetPubKey ( begin (), size ()))
        返回 假；
＃ENDIF
    返回 真；
}

bool  CPubKey::解压()
{
    如果(! IsValid ())
        返回 假；
# ifdef USE_SECP256K1
    int clen =大小（）；
    int ret = secp256k1_ecdsa_pubkey_decompress((unsigned char*)begin(), &clen);
    assert(ret);
    assert(clen == (int)size());
#else
    CECKey key;
    if (!key.SetPubKey(begin(), size()))
        return false;
    std::vector<unsigned char> pubkey;
    key.GetPubKey(pubkey, false);
    Set(pubkey.begin(), pubkey.end());
#endif
    return true;
}

bool CPubKey::Derive(CPubKey& pubkeyChild, unsigned char ccChild[32], unsigned int nChild, const unsigned char cc[32]) const
{
    assert(IsValid());
    assert((nChild >> 31) == 0);
    assert(begin() + 33 == end());
    unsigned char out[64];
    BIP32Hash(cc, nChild, *begin(), begin() + 1, out);
    memcpy(ccChild, out + 32, 32);
#ifdef USE_SECP256K1
    pubkeyChild = *this;
    bool ret = secp256k1_ecdsa_pubkey_tweak_add((unsigned char*)pubkeyChild.begin(), pubkeyChild.size(), out);
#else
    CECKey key;
    bool ret = key.SetPubKey(begin(), size());
    ret &= key.TweakPublic(out);
    std::vector<unsigned char> pubkey;
    key.GetPubKey(pubkey, true);
    pubkeyChild.Set(pubkey.begin(), pubkey.end());
#endif
    return ret;
}

void CExtPubKey::Encode(unsigned char code[74]) const
{
    code[0] = nDepth;
    memcpy(code + 1, vchFingerprint, 4);
    code[5] = (nChild >> 24) & 0xFF;
    code[6] = (nChild >> 16) & 0xFF;
    code[7] = (nChild >> 8) & 0xFF;
    code[8] = (nChild >> 0) & 0xFF;
    memcpy(code + 9, vchChainCode, 32);
    assert(pubkey.size() == 33);
    memcpy(code + 41, pubkey.begin(), 33);
}

void CExtPubKey::Decode(const unsigned char code[74])
{
    nDepth = code[0];
    memcpy(vchFingerprint, code + 1, 4);
    nChild = (code[5] << 24) | (code[6] << 16) | (code[7] << 8) | code[8];
    memcpy(vchChainCode, code + 9, 32);
    pubkey.Set(code + 41, code + 74);
}

bool CExtPubKey::Derive(CExtPubKey& out, unsigned int nChild) const
{
    出去。nDepth = nDepth + 1 ;
    CKeyID id = 公钥。获取ID ();
    memcpy (& out.vchFingerprint [ 0 ], &id, 4 );
    出去。nChild = nChild;
    返回公钥。派生( out.pubkey , out.vchChainCode , nChild, vchChainCode);
}
© 2021 GitHub, Inc.
条款
隐私
安全
地位
文档
联系 GitHub
价钱
应用程序接口
训练
博客
关于
