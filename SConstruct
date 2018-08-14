import os
import sys
import datetime
import excons

env = excons.MakeBaseEnv(output_dir="./scons-build")

libjpeg_srcs = excons.CollectFiles([".", "simd", "md5"], patterns=["*.c", "*.asm"], recursive=False)

def LibjpegName(static=False):
   libname = "jpeg"
   if sys.platform == "win32" and static:
      libname += "-static"
   return libname

def LibjpegPath(static=False):
   name = LibjpegName(static=static)
   if sys.platform == "win32":
      libname = name + ".lib"
   else:
      libname = "lib" + name + (".a" if static else excons.SharedLibraryLinkExt())
   return excons.OutputBaseDirectory() + "/lib/" + libname

def RequireLibjpeg(env, static=False):
   env.Append(CPPPATH=[excons.OutputBaseDirectory() + "/include"])
   env.Append(LIBPATH=[excons.OutputBaseDirectory() + "/lib"])
   excons.Link(env, LibjpegPath(static=static), static=static, force=True, silent=True)


libjpeg_outputs = ["include/jpeglib.h",
                   "include/turbojpeg.h",
                   "include/jconfig.h",
                   "include/jerror.h",
                   "include/jmorecfg.h",
                   LibjpegPath(True),
                   LibjpegPath(False)]

if sys.platform == "win32" and not os.path.isdir("./win/nasm-2.12.02"):
   import zipfile
   zf = zipfile.ZipFile("./win/nasm-2.12.02-win64.zip")
   zf.extractall("./win")

prjs = [{"name": "libjpeg",
         "type": "cmake",
         "cmake-opts": {"WITH_SIMD"      : excons.GetArgument("libjpeg-simd",       1, int),
                        "WITH_ARITH_ENC" : excons.GetArgument("libjpeg-arith-enc",  1, int),
                        "WITH_ARITH_DEC" : excons.GetArgument("libjpeg-arith-dec",  1, int),
                        "WITH_JPEG7"     : excons.GetArgument("libjpeg-jpeg7",      0, int),
                        "WITH_JPEG8"     : excons.GetArgument("libjpeg-jpeg8",      0, int),
                        "WITH_MEM_SRCDST": excons.GetArgument("libjpeg-mem-srcdst", 1, int),
                        "WITH_TURBOJPEG" : excons.GetArgument("libjpeg-turbojpeg",  1, int),
                        "WITH_12BIT"     : excons.GetArgument("libjpeg-12bit",      0, int),
                        "NASM"           : os.path.abspath("./win/nasm-2.12.02/nasm.exe"),
                        "WITH_CRT_DLL"   : 1},
         "cmake-cfgs": excons.CollectFiles([".", "simd", "md5"], patterns=["CMakeLists.txt"], recursive=False),
         "cmake-srcs": libjpeg_srcs,
         "cmake-outputs": libjpeg_outputs}]

excons.AddHelpOptions(libjpeg="""JPEG OPTIONS
  libjpeg-simd=0|1       : Include SIMD extensions [1]
  libjpeg-arith-enc=0|1  : Include arithmetic encoding support when emulating the libjpeg v6b API/ABI [1]
  libjpeg-arith-dec=0|1  : Include arithmetic decoding support when emulating the libjpeg v6b API/ABI [1]
  libjpeg-jpeg7=0|1      : Emulate libjpeg v7 API/ABI (this makes libjpeg-turbo backward incompatible with libjpeg v6b) [0]
  libjpeg-jpeg8=0|1      : Emulate libjpeg v8 API/ABI (this makes libjpeg-turbo backward incompatible with libjpeg v6b) [0]
  libjpeg-mem-srcdst=0|1 : Include in-memory source/destination manager functions when emulating the libjpeg v6b or v7 API/ABI [1]
  libjpeg-turbojpeg=0|1  : Include the TurboJPEG wrapper library and associated test programs [1]
  libjpeg-12bit=0|1      : Encode/decode JPEG images with 12-bit samples [0]
                           (implies with-simd=0 with-turbojpeg=0 with-arith-enc=0 with-arith-dec=0)""")

excons.DeclareTargets(env, prjs)

Export("LibjpegName LibjpegPath RequireLibjpeg")

