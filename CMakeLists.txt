cmake_minimum_required(VERSION 3.2)

set(CHIP bm1684)
set(PRODUCTFORM pcie)

set(MW_TOP /home/yuan/bitmain/work/middleware-soc CACHE INTERNAL "middleware-soc dir")
set(ARCH_LIST pcie)

add_subdirectory(bmvid)
add_subdirectory(bm_ffmpeg)

