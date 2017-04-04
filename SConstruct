import os
import sys
import datetime
import excons

env = excons.MakeBaseEnv(output_dir="./scons-build")

libjpeg_srcs = excons.CollectFiles([".", "simd", "md5"], patterns=["*.c", "*.asm", "CMakeLists.txt"], recursive=False)

if sys.platform == "win32":
   if not os.path.isdir("./win/nasm-2.12.02"):
      import zipfile
      zf = zipfile.ZipFile("./win/nasm-2.12.02-win64.zip")
      zf.extractall("./win")

   prjs = [{"name": "libjpeg",
            "type": "cmake",
            "cmake-opts": {"WITH_SIMD"      : excons.GetArgument("with-simd",       1, int),
                           "WITH_ARITH_ENC" : excons.GetArgument("with-arith-enc",  1, int),
                           "WITH_ARITH_DEC" : excons.GetArgument("with-arith-dec",  1, int),
                           "WITH_JPEG7"     : excons.GetArgument("with-jpeg7",      0, int),
                           "WITH_JPEG8"     : excons.GetArgument("with-jpeg8",      0, int),
                           "WITH_MEM_SRCDST": excons.GetArgument("with-mem-srcdst", 1, int),
                           "WITH_TURBOJPEG" : excons.GetArgument("with-turbojpeg",  1, int),
                           "WITH_12BIT"     : excons.GetArgument("with-12bit",      0, int),
                           "NASM"           : os.path.abspath("./win/nasm-2.12.02/nasm.exe")},
            "cmake-srcs": libjpeg_srcs}]

else:
   os.environ["CFLAGS"] = "-fPIC"

   prjs = [{"name": "libjpeg",
            "type": "automake",
            "automake-opts": {"--without-simd"      : excons.GetArgument("with-simd",      1, int) == 0,
                              "--without-arith-enc" : excons.GetArgument("with-arith-enc", 1, int) == 0,
                              "--without-arith-dec" : excons.GetArgument("with-arith-dec", 1, int) == 0,
                              "--with-jpeg7"        : excons.GetArgument("with-jpeg7",     0, int) != 0,
                              "--with-jpeg8"        : excons.GetArgument("with-jpeg8",     0, int) != 0,
                              "--without-mem-srcdst": excons.GetArgument("with-arith-enc", 1, int) == 0,
                              "--without-turbojpeg" : excons.GetArgument("with-arith-enc", 1, int) == 0,
                              "--with-12bit"        : excons.GetArgument("with-12bit",     0, int) != 0},
            "automake-srcs": libjpeg_srcs}]

excons.AddHelpOptions(libjpeg="""%s JPEG OPTIONS
  with-simd=0|1       : Include SIMD extensions [1]
  with-arith-enc=0|1  : Include arithmetic encoding support when emulating the libjpeg v6b API/ABI [1]
  with-arith-dec=0|1  : Include arithmetic decoding support when emulating the libjpeg v6b API/ABI [1]
  with-jpeg7=0|1      : Emulate libjpeg v7 API/ABI (this makes libjpeg-turbo backward incompatible with libjpeg v6b) [0]
  with-jpeg8=0|1      : Emulate libjpeg v8 API/ABI (this makes libjpeg-turbo backward incompatible with libjpeg v6b) [0]
  with-mem-srcdst=0|1 : Include in-memory source/destination manager functions when emulating the libjpeg v6b or v7 API/ABI [1]
  with-turbojpeg=0|1  : Include the TurboJPEG wrapper library and associated test programs [1]
  with-12bit=0|1      : Encode/decode JPEG images with 12-bit samples [0]
                        (implies with-simd=0 with-turbojpeg=0 with-arith-enc=0 with-arith-dec=0)""" % prjs[0]["type"].upper())
excons.DeclareTargets(env, prjs)

# ==============================================================================

def RequireLibjpeg(env, static=False):
   env.Append(CPPPATH=[excons.OutputBaseDirectory() + "/include"])
   env.Append(LIBPATH=[excons.OutputBaseDirectory() + "/lib"])
   # Any defines?
   if static:
      if sys.platform == "win32":
         env.Append(LIBS=["jpeg-static"])
      else:
         if not excons.StaticallyLink(env, "jpeg", silent=True):
            env.Append(LIBS=["jpeg"])
   else:
      env.Append(LIBS=["jpeg"])

def LibjpegName(static=False):
   if sys.platform == "win32":
      basename = ("jpeg-static" if static else "jpeg") + ".lib"
   else:
      basename = "libjpeg" + (".a" if static else ".so")
   return excons.OutputBaseDirectory() + "/lib/" + basename

Export("RequireLibjpeg LibjpegName")

