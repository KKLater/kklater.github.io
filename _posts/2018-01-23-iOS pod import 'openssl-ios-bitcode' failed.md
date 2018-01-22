---
layout:         post
title:          pod 导入 'openssl-ios-bitcode' 失败
subtitle:       
location:  北京 中国   
date:           2018-1-23 00:41:00
categories: iOS 
tags:  iOS Cocoapods
excerpt:  pod 导入 'openssl-ios-bitcode' 失败，导致项目不能正常运行
---

## 问题描述

项目的 Podfile 文件中包含有 'pod 'openssl-ios-bitcode', '~> 1.0.212'' ， ’pod update —no-repo-update'，报错。

```shell
localhost:OpenSSLTest Later$ pod update --no-repo-update
Update all pods
Analyzing dependencies
Downloading dependencies
Installing openssl-ios-bitcode (1.0.212)
[!] /bin/bash -c 
set -e
VERSION="1.0.2l"
SDKVERSION=`xcrun --sdk iphoneos --show-sdk-version 2> /dev/null`
MIN_SDKVERSION="7.0"

BASEPATH="${PWD}"
CURRENTPATH="${TMPDIR}/openssl"
ARCHS="i386 x86_64 armv7 arm64"
DEVELOPER=`xcode-select -print-path`

mkdir -p "${CURRENTPATH}"
mkdir -p "${CURRENTPATH}/bin"

cp "file.tgz" "${CURRENTPATH}/file.tgz"
cd "${CURRENTPATH}"
tar -xzf file.tgz
cd "openssl-${VERSION}"

for ARCH in ${ARCHS}
do
  CONFIGURE_FOR="iphoneos-cross"

  if [ "${ARCH}" == "i386" ] || [ "${ARCH}" == "x86_64" ] ;
  then
    PLATFORM="iPhoneSimulator"
    if [ "${ARCH}" == "x86_64" ] ;
    then
      CONFIGURE_FOR="darwin64-x86_64-cc"
    fi
  else
    sed -ie "s!static volatile sig_atomic_t intr_signal;!static volatile intr_signal;!" "crypto/ui/ui_openssl.c"
    PLATFORM="iPhoneOS"
  fi

  export CROSS_TOP="${DEVELOPER}/Platforms/${PLATFORM}.platform/Developer"
  export CROSS_SDK="${PLATFORM}${SDKVERSION}.sdk"

  echo "Building openssl-${VERSION} for ${PLATFORM} ${SDKVERSION} ${ARCH}"
  echo "Please stand by..."

  export CC="${DEVELOPER}/usr/bin/gcc -arch ${ARCH} -miphoneos-version-min=${MIN_SDKVERSION} -fembed-bitcode"
  mkdir -p "${CURRENTPATH}/bin/${PLATFORM}${SDKVERSION}-${ARCH}.sdk"
  LOG="${CURRENTPATH}/bin/${PLATFORM}${SDKVERSION}-${ARCH}.sdk/build-openssl-${VERSION}.log"

  LIPO_LIBSSL="${LIPO_LIBSSL} ${CURRENTPATH}/bin/${PLATFORM}${SDKVERSION}-${ARCH}.sdk/lib/libssl.a"
  LIPO_LIBCRYPTO="${LIPO_LIBCRYPTO} ${CURRENTPATH}/bin/${PLATFORM}${SDKVERSION}-${ARCH}.sdk/lib/libcrypto.a"

  ./Configure ${CONFIGURE_FOR} --openssldir="${CURRENTPATH}/bin/${PLATFORM}${SDKVERSION}-${ARCH}.sdk" > "${LOG}" 2>&1
  sed -ie "s!^CFLAG=!CFLAG=-isysroot ${CROSS_TOP}/SDKs/${CROSS_SDK} !" "Makefile"

  make >> "${LOG}" 2>&1
  make all install_sw >> "${LOG}" 2>&1
  make clean >> "${LOG}" 2>&1
done


echo "Build library..."
rm -rf "${BASEPATH}/lib/"
mkdir -p "${BASEPATH}/lib/"
lipo -create ${LIPO_LIBSSL}    -output "${BASEPATH}/lib/libssl.a"
lipo -create ${LIPO_LIBCRYPTO} -output "${BASEPATH}/lib/libcrypto.a"

echo "Copying headers..."
rm -rf "${BASEPATH}/opensslIncludes/"
mkdir -p "${BASEPATH}/opensslIncludes/"
cp -RL "${CURRENTPATH}/openssl-${VERSION}/include/openssl" "${BASEPATH}/opensslIncludes/"

cd "${BASEPATH}"
echo "Building done."

echo "Cleaning up..."
rm -rf "${CURRENTPATH}"
echo "Done."

Building openssl-1.0.2l for iPhoneSimulator 11.1 i386
Please stand by...

localhost:OpenSSLTest Later$ 
```

## 解决方案

```shell
 rm -r ${TMPDIR}/openssl/
```

![](https://raw.githubusercontent.com/KKLater/kklater.github.io/master/_posts/_image/6757D235-7D85-4AFC-A578-D02D05D190D3.png)

参考资料:[https://github.com/FredericJacobs/OpenSSL-Pod/issues/32](https://github.com/FredericJacobs/OpenSSL-Pod/issues/32)

## 问题原因

思考中，有知道的给告诉下。