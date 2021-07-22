cmake_minimum_required(VERSION 3.2)
set(BMVID_TOP ${MW_TOP}/bmvid)

# ION
set(BMION_INC ${BMVID_TOP}/allocator/ion/inc)

if ("${CHIP}" STREQUAL bm1684)
    add_definitions(-DCHIP_BM1684=1 -DUSE_ION_MEMORY=1 -DBM_ION_MEM=1 -D_FILE_OFFSET_BITS=64 -D_LARGEFILE_SOURCE
            -DCHIP_BM1684=1 -DASIC_EVB)
endif()

if ("${PRODUCTFORM}" STREQUAL pcie)
    add_definitions(-DBM_PCIE_MODE=1)
endif()

add_definitions(-DLIBBMJPULITE=1)


add_library(bmion ${BMVID_TOP}/allocator/ion/src/bm_ion.c
        ${BMVID_TOP}/allocator/ion/src/bm_pcie_buffer.c
        ${BMVID_TOP}/allocator/ion/src/bmionapi.c)
target_include_directories(bmion PRIVATE ${BMION_INC})

add_executable(test_ion ${BMVID_TOP}/allocator/ion/example/test_ion.c)
target_link_libraries(test_ion bmion)
target_include_directories(test_ion PRIVATE ${BMION_INC})

# jpeg

set(JPEG_TOP ${BMVID_TOP}/jpeg/driver)
## bmjpulite
set(BMJPULITE_TOP ${JPEG_TOP}/bmjpulite)
set(JPUAPI_TOP ${JPEG_TOP}/jpuapi)
set(jpuapi_srcs ${JPUAPI_TOP}/jpuapi.c
        ${JPUAPI_TOP}/jpuapifunc.c
        )

include_directories(${BMJPULITE_TOP}/inc)
set(bmjpulite_srcs ${BMJPULITE_TOP}/src/jpu_io.c
        ${BMJPULITE_TOP}/src/jpu_lib.c
        ${BMJPULITE_TOP}/src/jpu_lib_dec.c
        ${BMJPULITE_TOP}/src/jpu_lib_enc.c
        ${BMJPULITE_TOP}/src/jpu_logging.c
        ${BMJPULITE_TOP}/src/jpulog.c
        ${JPEG_TOP}/jdi/linux/jdi.c
        ${JPEG_TOP}/jdi/mm.c
        ${jpuapi_srcs}
        )
add_library(bmjpulite ${bmjpulite_srcs})
target_include_directories(bmjpulite PRIVATE ${JPUAPI_TOP} ${JPEG_TOP}/include ${BMION_INC})

## bmjpuapi
set(BMJPUAPI_TOP ${JPEG_TOP}/bmjpuapi)

set(bmjpuapi_srcs ${BMJPUAPI_TOP}/src/bmjpuapi.c
        ${BMJPUAPI_TOP}/src/bmjpuapi.c
        ${BMJPUAPI_TOP}/src/bmjpuapi_jpulib.c
        ${BMJPUAPI_TOP}/src/bmjpuapi_jpeg.c
        ${BMJPUAPI_TOP}/src/bmjpuapi_parse_jpeg.c)
add_library(bmjpuapi SHARED ${bmjpuapi_srcs})
target_include_directories(bmjpuapi PRIVATE ${JPEG_TOP}/include ${BMJPUAPI_TOP}/inc ${BMION_INC})

set(jpeg_example_inc_dirs ${JPEG_TOP}/include ${BMJPUAPI_TOP}/inc ${BMION_INC})
## bmjpegdec
add_executable(bmjpegdec ${BMJPUAPI_TOP}/example/jpeg_dec.c
        ${BMJPUAPI_TOP}/example/jpeg_common.c)
target_link_libraries(bmjpegdec bmjpuapi bmjpulite bmion pthread)
target_include_directories(bmjpegdec PRIVATE ${jpeg_example_inc_dirs})

## bmjpegenc
add_executable(bmjpegenc ${BMJPUAPI_TOP}/example/jpeg_enc.c
        ${BMJPUAPI_TOP}/example/jpeg_common.c)
target_link_libraries(bmjpegenc bmjpuapi bmjpulite bmion pthread)
target_include_directories(bmjpegenc PRIVATE ${jpeg_example_inc_dirs})

## bmjpegmulti
add_executable(bmjpegmulti ${BMJPUAPI_TOP}/example/jpeg_multi.c
        ${BMJPUAPI_TOP}/example/jpeg_common.c
        ${BMJPUAPI_TOP}/example/bmjpurun.c
        )
target_link_libraries(bmjpegmulti bmjpuapi bmjpulite bmion pthread)
target_include_directories(bmjpegmulti PRIVATE ${jpeg_example_inc_dirs})

#VPU

## bmvpulite
set(BMVPULITE_TOP ${BMVID_TOP}/cnm/driver/mmf/bmvpulite)
add_definitions(-DBM_ASIC=1)

add_library(bmvpulite SHARED ${BMVPULITE_TOP}/src/common.c
        ${BMVPULITE_TOP}/src/logging.c
        ${BMVPULITE_TOP}/src/enc.c
        ${BMVPULITE_TOP}/src/vdi.c
        )
target_include_directories(bmvpulite PRIVATE ${BMVPULITE_TOP}/inc ${BMVID_TOP}/cnm/driver ${BMION_INC}
        ${BMVID_TOP}/cnm/driver/vdi/linux/driver)

## bmvpuapi
set(BMVPUAPI_TOP ${BMVID_TOP}/cnm/driver/mmf/bmvpuapi)

add_library(bmvpuapi SHARED ${BMVPUAPI_TOP}/src/allocator.c
        ${BMVPUAPI_TOP}/src/encoder.c
        ${BMVPUAPI_TOP}/src/queue.c
        ${BMVPUAPI_TOP}/src/misc.c
        ${BMVPUAPI_TOP}/src/log.c
        )
target_include_directories(bmvpuapi PRIVATE ${BMVPUAPI_TOP}/inc ${BMVPULITE_TOP}/inc ${BMION_INC})

## bmvideo
set(BMVIDEO_TOP ${BMVID_TOP}/cnm/driver/mmf/bm_video_interface)
set(BMVIDEO_DRIVER_TOP ${BMVID_TOP}/cnm/driver)

set(bmvideo_driver_srcs ${BMVIDEO_DRIVER_TOP}/sample_v2/helper/main_helper.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/helper/vpuhelper.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/helper/bitstream/bitstreamfeeder.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/helper/bitstream/bitstreamreader.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/helper/bitstream/bsfeeder_fixedsize_impl.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/helper/bitstream/bsfeeder_framesize_impl.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/helper/bitstream/bsfeeder_size_plus_es_impl.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/helper/comparator/bin_comparator_impl.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/helper/comparator/comparator.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/helper/comparator/md5_comparator_impl.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/helper/comparator/yuv_comparator_impl.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/helper/misc/cfgParser.c

        ${BMVIDEO_DRIVER_TOP}/sample_v2/helper/misc/cnm_video_helper.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/helper/misc/container.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/helper/misc/datastructure.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/helper/misc/debug.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/helper/misc/bw_monitor.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/helper/misc/pf_monitor.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/helper/misc/md5.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/helper/yuv/yuvfeeder.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/helper/yuv/yuvLoaderfeeder.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/helper/yuv/yuvCfbcfeeder.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/component/cnm_app.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/component/cnm_task.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/component/component.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/component_encoder/encoder_listener.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/component_encoder/component_enc_encoder.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/component_encoder/component_enc_feeder.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/component_encoder/component_enc_reader.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/component_decoder/component_dec_decoder.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/component_decoder/component_dec_feeder.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/component_decoder/component_dec_renderer.c
        ${BMVIDEO_DRIVER_TOP}/sample_v2/component_decoder/decoder_listener.c
        ${BMVIDEO_DRIVER_TOP}/vpuapi/product.c
        ${BMVIDEO_DRIVER_TOP}/vpuapi/vpuapifunc.c
        ${BMVIDEO_DRIVER_TOP}/vpuapi/vpuapi.c
        ${BMVIDEO_DRIVER_TOP}/vpuapi/coda9//coda9.c
        ${BMVIDEO_DRIVER_TOP}/vpuapi/wave/wave5.c
        ${BMVIDEO_DRIVER_TOP}/vdi/linux/vdi.c
        ${BMVIDEO_DRIVER_TOP}/vdi/linux/vdi_osal.c
        ${BMVIDEO_DRIVER_TOP}/vdi/mm.c
        )

add_library(bmvideo SHARED ${BMVIDEO_TOP}/src/bm_video_interface.c
        ${BMVIDEO_TOP}/src/bm_video_syscxt.c ${bmvideo_driver_srcs})
set(bmvideo_inc
        ${BMVID_TOP}/cnm/driver/vpuapi
        ${BMVID_TOP}/cnm/driver/sample_v2/helper
        ${BMVID_TOP}/cnm/driver/sample_v2/helper/misc
        ${BMVID_TOP}/cnm/driver/sample_v2/component
        ${BMVID_TOP}/cnm/driver/vdi
        ${BMVIDEO_TOP}/inc
        ${BMVID_TOP}/cnm/driver/sample_v2/component_decoder
        ${BMVID_TOP}/cnm/driver/sample_v2/component_encoder
        ${BMVID_TOP}/cnm/driver
        ${BMION_INC})
message(STATUS "bmvideo inc=${bmvideo_inc}")
target_include_directories(bmvideo PRIVATE ${bmvideo_inc})
## vpureset
add_executable(vpureset ${BMVIDEO_TOP}/test/vpu_reset.c)
target_include_directories(vpureset PRIVATE ${bmvideo_inc})
target_link_libraries(vpureset bmvideo bmion pthread dl)
## vpuinfo
add_executable(vpuinfo ${BMVIDEO_TOP}/test/vpu_info.c)
target_include_directories(vpuinfo PRIVATE ${bmvideo_inc})
target_link_libraries(vpuinfo bmvideo bmion pthread dl)
## bm_test
add_executable(bm_test ${BMVIDEO_TOP}/test/bm_test.c)
target_include_directories(bm_test PRIVATE ${bmvideo_inc})
target_link_libraries(bm_test bmvideo bmion pthread dl)


#vpp
set(BMVPP_TOP ${BMVID_TOP}/vpp)
add_definitions(-DPCIE_MODE=1 -DION_CACHE=1)
add_library(bmvpp SHARED ${BMVPP_TOP}/driver/src/vppion.c
        ${BMVPP_TOP}/driver/src/bm1684/vpplib.c
        )
target_include_directories(bmvpp PRIVATE ${BMVPP_TOP}/driver/include
        ${BMVPP_TOP}/driver/include/bm1684
        ${MW_TOP}/bm_opencv/3rdparty/libbmcv/include)

add_library(bmvppapi SHARED ${BMVPP_TOP}/bmvppapi/src/common.c
        ${BMVPP_TOP}/bmvppapi/src/bmvpp1684.c)
target_include_directories(bmvppapi PRIVATE ${BMVPP_TOP}/bmvppapi/inc)
