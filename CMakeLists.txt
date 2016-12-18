cmake_minimum_required(VERSION 2.8)
project(7kaa)


include(CheckSymbolExists)
include(CheckIncludeFiles)
include(CheckTypeSize)

CHECK_SYMBOL_EXISTS(gettext "libintl.h" SEVENKAA_HAVE_GETTEXT)
CHECK_SYMBOL_EXISTS(dcgettext "libintl.h" SEVENKAA_HAVE_DCGETTEXT)
CHECK_SYMBOL_EXISTS(iconv "iconv.h" SEVENKAA_HAVE_ICONV)

CHECK_INCLUDE_FILES("inttypes.h"  SEVENKAA_HAVE_INTTYPES_H)
CHECK_INCLUDE_FILES("memory.h"    SEVENKAA_HAVE_MEMORY_H)
CHECK_INCLUDE_FILES("stdint.h"    SEVENKAA_HAVE_STDINT_H)
CHECK_INCLUDE_FILES("stdlib.h"    SEVENKAA_HAVE_STDLIB_H)
CHECK_INCLUDE_FILES("strings.h"   SEVENKAA_HAVE_STRINGS_H)
CHECK_INCLUDE_FILES("string.h"    SEVENKAA_HAVE_STRING_H)
CHECK_INCLUDE_FILES("sys/stat.h"  SEVENKAA_HAVE_SYS_STAT_H)
CHECK_INCLUDE_FILES("sys/types.h" SEVENKAA_HAVE_SYS_TYPES_H)
CHECK_INCLUDE_FILES("unistd.h"    SEVENKAA_HAVE_UNISTD_H)

CHECK_TYPE_SIZE("char"      SEVENKAA_SIZEOF_CHAR)
CHECK_TYPE_SIZE("short"     SEVENKAA_SIZEOF_SHORT)
CHECK_TYPE_SIZE("int"       SEVENKAA_SIZEOF_INT)
CHECK_TYPE_SIZE("long"      SEVENKAA_SIZEOF_LONG)
CHECK_TYPE_SIZE("long long" SEVENKAA_SIZEOF_LONG_LONG)
CHECK_TYPE_SIZE("void*"     SEVENKAA_SIZEOF_VOID_PTR)



set(SEVENKAA_BASEDIR "${CMAKE_CURRENT_SOURCE_DIR}")
set(SEVENKAA_INCLUDE_DIR "${SEVENKAA_BASEDIR}/include/")
set(SEVENKAA_INCLUDE_TEMPLATE "${SEVENKAA_BASEDIR}/config.h.cmake.in")

set(SEVENKAA_CONFIG_OUTPUT_DIR "${CMAKE_BINARY_DIR}/include")
if(NOT EXISTS "${SEVENKAA_CONFIG_OUTPUT_DIR}")
  file(MAKE_DIRECTORY "${SEVENKAA_CONFIG_OUTPUT_DIR}")
endif()
set(SEVENKAA_COMFIG_INCLUDE_FILE "${SEVENKAA_CONFIG_OUTPUT_DIR}/config.h")

set(SEVENKAA_VERSION_TAG "git-dev-build" CACHE STRING "The version number/tag of the build")
set(SEVENKAA_PROGRAM_NAME "7kaa")

set(SEVENKAA_VERSION "\"${SEVENKAA_VERSION_TAG}\"")
set(SEVENKAA_PACKAGE "\"${SEVENKAA_PROGRAM_NAME}\"")
set(SEVENKAA_PACKAGE_BUGREPORT "\"skfans-devel@lists.sourceforge.net\"")
set(SEVENKAA_PACKAGE_DATA_PATH "\"${CMAKE_INSTALL_PREFIX}/share/${SEVENKAA_PROGRAM_NAME}\"")
set(SEVENKAA_PACKAGE_NAME "\"${SEVENKAA_PROGRAM_NAME}\"")
set(SEVENKAA_PACKAGE_STRING "\"${SEVENKAA_PROGRAM_NAME} ${SEVENKAA_VERSION_TAG}\"")
set(SEVENKAA_PACKAGE_TARNAME "\"${SEVENKAA_PROGRAM_NAME}\"")
set(SEVENKAA_PACKAGE_URL "\"http://7kfans.com/\"")
set(SEVENKAA_PACKAGE_VERSION "${SEVENKAA_VERSION}")

option(SEVENKAA_ENABLE_NLS "Translate program messages to the user's native language" OFF)
option(SEVENKAA_DEBUG "Enable additional debugging features/checks" OFF)
option(SEVENKAA_DYNARRAY_DEBUG_ELEMENT_ACCESS "Enable element access checking for dynamic arrays" OFF)
option(SEVENKAA_USE_SDL "Use SDL2 as the graphics rendering/output and mouse/keyboard input engine" ON)
option(SEVENKAA_USE_OPENAL "Use OpenAL as the sound output engine" ON)
option(SEVENKAA_USE_ASM "Do use (legacy) assembler code (asm is windows only)" OFF)

mark_as_advanced(SEVENKAA_USE_NOINPUT SEVENKAA_USE_NOVIDEO)


include(FindPkgConfig)
PKG_SEARCH_MODULE(SDL2 sdl2)

include(FindOpenAL)
set(SEVENKAA_OPENAL_ALC_H "<AL/alc.h>" CACHE STRING "Define to the equivalent of <AL/alc.h> on your system")
set(SEVENKAA_OPENAL_AL_H "<AL/al.h>" CACHE STRING "Define to the equivalent of <AL/al.h> on your system")

  
function(_SEVENKAA_find_enet)
  find_file(SEVENKAA_ENET_HEADER "enet/enet.h")
  find_library(SEVENKAA_ENET_LIBRARY "enet")
endfunction(_SEVENKAA_find_enet)
_SEVENKAA_find_enet()

function(_SEVENKAA_find_type_of_size OUTVARNAME TYPE_SIZE)
  if    (TYPE_SIZE EQUAL SEVENKAA_SIZEOF_CHAR)
    set(${OUTVARNAME} "char" PARENT_SCOPE)
  elseif(TYPE_SIZE EQUAL SEVENKAA_SIZEOF_SHORT)
    set(${OUTVARNAME} "short" PARENT_SCOPE)
  elseif(TYPE_SIZE EQUAL SEVENKAA_SIZEOF_INT)
    set(${OUTVARNAME} "int" PARENT_SCOPE)
  elseif(TYPE_SIZE EQUAL SEVENKAA_SIZEOF_LONG)
    set(${OUTVARNAME} "long" PARENT_SCOPE)
  elseif(TYPE_SIZE EQUAL SEVENKAA_SIZEOF_LONG_LONG)
    set(${OUTVARNAME} "long long" PARENT_SCOPE)
  endif()
endfunction(_SEVENKAA_find_type_of_size)

macro(_SEVENKAA_set_01 VARNAME VALUE)
  if("${VALUE}")
    #message("${VARNAME} ${VALUE} -> 1")
    set(${VARNAME} 1)
  else()
    #message("${VARNAME} ${VALUE} -> 0")
    set(${VARNAME} 0)
  endif()
endmacro(_SEVENKAA_set_01)

function(_SEVENKAA_make_config_h)
  #scoped to not spill variables
  _SEVENKAA_set_01(DEBUG            "${SEVENKAA_DEBUG}")
  _SEVENKAA_set_01(DYNARRAY_DEBUG_ELEMENT_ACCESS "${SEVENKAA_DYNARRAY_DEBUG_ELEMENT_ACCESS}")
  _SEVENKAA_set_01(ENABLE_NLS       "${SEVENKAA_ENABLE_NLS}")
  _SEVENKAA_set_01(USE_SDL          "${SEVENKAA_USE_SDL}")
  _SEVENKAA_set_01(USE_OPENAL       "${SEVENKAA_USE_OPENAL}")
  
  _SEVENKAA_set_01(HAVE_DCGETTEXT   "${SEVENKAA_HAVE_DCGETTEXT}")
  _SEVENKAA_set_01(HAVE_GETTEXT     "${SEVENKAA_HAVE_GETTEXT}")
  _SEVENKAA_set_01(HAVE_ICONV       "${SEVENKAA_HAVE_ICONV}")
  _SEVENKAA_set_01(HAVE_INTTYPES_H  "${SEVENKAA_HAVE_INTTYPES_H}")
  _SEVENKAA_set_01(HAVE_MEMORY_H    "${SEVENKAA_HAVE_MEMORY_H}")
  _SEVENKAA_set_01(HAVE_STDINT_H    "${SEVENKAA_HAVE_STDINT_H}")
  _SEVENKAA_set_01(HAVE_STDLIB_H    "${SEVENKAA_HAVE_STDLIB_H}")
  _SEVENKAA_set_01(HAVE_STRINGS_H   "${SEVENKAA_HAVE_STRINGS_H}")
  _SEVENKAA_set_01(HAVE_STRING_H    "${SEVENKAA_HAVE_STRING_H}")
  _SEVENKAA_set_01(HAVE_SYS_STAT_H  "${SEVENKAA_HAVE_SYS_STAT_H}")
  _SEVENKAA_set_01(HAVE_SYS_TYPES_H "${SEVENKAA_HAVE_SYS_TYPES_H}")
  _SEVENKAA_set_01(HAVE_UNISTD_H    "${SEVENKAA_HAVE_UNISTD_H}")
  
  if(WIN32)
    set(NO_WINDOWS 0)
  else()
    set(NO_WINDOWS 1)
  endif()
  
  if(SEVENKAA_USE_ASM AND WIN32)
    set(USE_ASM 1)
  else()
    set(USE_ASM 0)
  endif()
  
  if(NOT SEVENKAA_HAVE_STDINT_H)
    _SEVENKAA_find_type_of_size(int8_t 1)
    _SEVENKAA_find_type_of_size(int16_t 2)
    _SEVENKAA_find_type_of_size(int32_t 4)
    _SEVENKAA_find_type_of_size(int64_t 8)
    _SEVENKAA_find_type_of_size(INTPTR_T ${SEVENKAA_SIZEOF_VOID_PTR})
    
    set(uint8_t "unsigned ${int8_t}")
    set(uint16_t "unsigned ${int16_t}")
    set(uint32_t "unsigned ${int32_t}")
    set(uint64_t "unsigned ${int64_t}")
    set(size_t "unsigned ${INTPTR_T}")
  endif()
  
  if(SEVENKAA_USE_OPENAL)
    set(OPENAL_ALC_H "${SEVENKAA_OPENAL_ALC_H}")
    set(OPENAL_AL_H "${SEVENKAA_OPENAL_AL_H}")
  endif()
  
  configure_file("${SEVENKAA_INCLUDE_TEMPLATE}" "${SEVENKAA_COMFIG_INCLUDE_FILE}")
endfunction(_SEVENKAA_make_config_h)
_SEVENKAA_make_config_h()


function(_SEVENKAA_enable_cxx11)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=gnu++11" PARENT_SCOPE)
endfunction(_SEVENKAA_enable_cxx11)
#_SEVENKAA_enable_cxx11()

set(H_FILES
  include/ALL.h
  include/asmfun.h
  include/audio_base.h
  include/audio_stream.h
  include/c99_printf.h
  include/COLCODE.h
  include/COLOR.h
  include/CRC.h
  include/dbglog.h
  include/file_input_stream.h
  include/file_io_visitor.h
  include/file_output_stream.h
  include/file_reader.h
  include/file_util.h
  include/file_writer.h
  include/GAMEDEF.h
  include/gettext-gnu.h
  include/gettext.h
  include/IMGFUN.h
  include/input_stream.h
  include/KEY.h
  include/mem_input_stream.h
  include/misc_uuid.h
  include/mouse_sdl.h
  include/MPTYPES.h
  include/multiplayer.h
  include/OANLINE.h
  include/OAUDIO.h
  include/OBATTLE.h
  include/OB_FLAME.h
  include/OB_HOMIN.h
  include/OBLOB.h
  include/OBOX.h
  include/OB_PROJ.h
  include/OBULLET.h
  include/OBUTT3D.h
  include/OBUTTCUS.h
  include/OBUTTON.h
  include/OCOLTBL.h
  include/OCONFIG.h
  include/OCRC_STO.h
  include/ODATE.h
  include/ODB.h
  include/ODIR.h
  include/ODYNARRB.h
  include/ODYNARR.h
  include/OEFFECT.h
  include/OERRCTRL.h
  include/OERROR.h
  include/OEXPMASK.h
  include/OF_BASE.h
  include/OF_CAMP.h
  include/OF_FACT.h
  include/OF_HARB.h
  include/OFILE.h
  include/OFILETXT.h
  include/OF_INN.h
  include/OFIRERES.h
  include/OFIRMA.h
  include/OFIRMALL.h
  include/OFIRMDIE.h
  include/OFIRM.h
  include/OFIRMID.h
  include/OFIRMRES.h
  include/OFLAME.h
  include/OF_MARK.h
  include/OF_MINE.h
  include/OF_MONS.h
  include/OFONT.h
  include/OF_RESE.h
  include/OF_WAR.h
  include/OGAME.h
  include/OGAMESET.h
  include/OGETA.h
  include/OGET.h
  include/OGFILE.h
  include/OGF_V1.h
  include/OGODRES.h
  include/OGRPSEL.h
  include/OHELP.h
  include/OHILLRES.h
  include/OHSETRES.h
  include/OIMGRES.h
  include/OINFO.h
  include/OINGMENU.h
  include/OISOAREA.h
  include/OLIGHTN.h
  include/OLOG.h
  include/OLONGLOG.h
  include/OLZW.h
  include/OMATRIX.h
  include/OMISC.h
  include/OMLINK.h
  include/OMONSRES.h
  include/OMOUSE2.h
  include/OMOUSECR.h
  include/OMOUSE.h
  include/OMUSIC.h
  include/ONATIONA.h
  include/ONATIONB.h
  include/ONATION.h
  include/ONEWS.h
  include/OOPTMENU.h
  include/openal_audio.h
  include/OPLANT.h
  include/OPLASMA.h
  include/OPOWER.h
  include/ORACERES.h
  include/ORAIN.h
  include/ORAWRES.h
  include/OREBEL.h
  include/OREGION.h
  include/OREGIONS.h
  include/OREMOTE.h
  include/OREMOTEQ.h
  include/ORESDB.h
  include/ORES.h
  include/ORESTXT.h
  include/ORESX.h
  include/OROCK.h
  include/OROCKRES.h
  include/OSCROLL.h
  include/OSE.h
  include/OSERES.h
  include/OSFRMRES.h
  include/OSITE.h
  include/OSKILL.h
  include/OSLIDCUS.h
  include/OSLIDER.h
  include/OSNOWG.h
  include/OSNOW.h
  include/OSNOWRES.h
  include/OSPATH.h
  include/OSPINNER.h
  include/OSPREUSE.h
  include/OSPRITE.h
  include/OSPRTRES.h
  include/OSPY.h
  include/OSTR.h
  include/OSYS.h
  include/OTALKMSG.h
  include/OTALKRES.h
  include/OTARRAY.h
  include/OTECHRES.h
  include/OTERRAIN.h
  include/OTORNADO.h
  include/OTOWN.h
  include/OTownNetwork.h
  include/OTOWNREC.h
  include/OTOWNRES.h
  include/OTRANSL.h
  include/OTUTOR.h
  include/OU_CARA.h
  include/OU_CART.h
  include/OU_GOD.h
  include/OU_MARI.h
  include/OU_MONS.h
  include/OUNITALL.h
  include/OUNIT.h
  include/OUNITRES.h
  include/output_stream.h
  include/OU_VEHI.h
  include/OVBROWIF.h
  include/OVBROWSE.h
  include/OVGABUF.h
  include/OVGA.h
  include/OVGALOCK.h
  include/OVIDEO.h
  include/OVOLUME.h
  include/OVQUEUE.h
  include/OWALLRES.h
  include/OWARPT.h
  include/OWEATHER.h
  include/OWORLD.h
  include/OWORLDMT.h
  include/player_desc.h
  include/RESOURCE.h
  include/surface_base.h
  include/surface.h
  include/surface_sdl.h
  include/version.h
  include/vga_base.h
  include/vga_sdl.h
  include/vga_util.h
  include/WALLTILE.h
  include/wav_stream.h
  include/win32_compat.h
)

set(CPP_FILES
  src/OUNITM.cpp
  src/OGAMMAIN.cpp
  src/OAI_TALK.cpp
  src/OW_PLANT.cpp
  src/OAI_INFO.cpp
  src/OAI_QUER.cpp
  src/OAI_MONS.cpp
  src/OSPRITEA.cpp
  src/OF_HARB2.cpp
  src/OU_MARIS.cpp
  src/OUNITAI.cpp
  src/OUNITD.cpp
  src/OR_AI.cpp
  src/OBULLET.cpp
  src/OREGIONS.cpp
  src/OUNIT.cpp
  src/OU_MARIF.cpp
  src/file_output_stream.cpp
  src/ORAWRES.cpp
  src/OSYS3.cpp
  src/OTownNetwork.cpp
  src/OW_SOUND.cpp
  src/OUNITAM.cpp
  src/OTRANSL.cpp
  src/OWORLD.cpp
  src/OF_MINE.cpp
  src/file_reader.cpp
  src/OAI_TOWN.cpp
  src/OREMOTEM.cpp
  src/OTALKENG.cpp
  src/OOPTMENU.cpp
  src/OAI_ATTK.cpp
  src/OREMOTEQ.cpp
  src/OGFILE.cpp
  src/OBULLETA.cpp
  src/OUNIT2.cpp
  src/OVBROWSE.cpp
  src/OR_ECO.cpp
  src/OW_FIRE.cpp
  src/OVQUEUE.cpp
  src/OF_MARK.cpp
  src/ODATE.cpp
  src/OTALKFRE.cpp
  src/OGAMENCY.cpp
  src/OB_PROJ.cpp
  src/OSPRESMO.cpp
  src/OGFILEA.cpp
  src/OSITE.cpp
  src/OGAMESET.cpp
  src/OVOLUME.cpp
  src/OSYS2.cpp
  src/OUNITI.cpp
  src/ONATIONA.cpp
  src/OB_HOMIN.cpp
  src/OROCKRES.cpp
  src/OFIRMDIE.cpp
  src/OUNITATB.cpp
  src/OSPREUSE.cpp
  src/OLONGLOG.cpp
  src/OTOWNDRW.cpp
  src/OTOWNA.cpp
  src/OUNITAT3.cpp
  src/OTUTOR.cpp
  src/vgautil.cpp
  src/vgautil2.cpp
  src/OTOWN.cpp
  src/OGAMSCE2.cpp
  src/OPLASMA.cpp
  src/OREMOTE.cpp
  src/OERROR.cpp
  src/OBUTTCUS.cpp
  src/OLOG.cpp
  src/OMOUSE.cpp
  src/OMEM.cpp
  src/OF_MARK2.cpp
  src/OWARPT.cpp
  src/input_stream.cpp
  src/OAI_TRAD.cpp
  src/OINFO.cpp
  src/ONEWSENG.cpp
  src/OWORLD_M.cpp
  src/OTUTOR2.cpp
  src/ORAIN1.cpp
  src/OU_GOD.cpp
  src/ONEWSGER.cpp
  src/OCONFIG.cpp
  src/enet/multiplayer.cpp
  src/OAI_GRAN.cpp
  src/OSNOW2.cpp
  src/OW_ROCK.cpp
  src/OTECHRES.cpp
  src/ORESDB.cpp
  src/OANLINE.cpp
  src/OINGMENU.cpp
  src/OSFRMRES.cpp
  src/OR_MIL.cpp
  src/OFIRMIF.cpp
  src/OUNITAC.cpp
  src/OF_WAR2.cpp
  src/ORACERES.cpp
  src/OF_WAR.cpp
  src/OU_CARA.cpp
  src/OU_CARAS.cpp
  src/OEXPMASK.cpp
  src/OF_FACT2.cpp
  src/OUNITS.cpp
  src/OTALKRES.cpp
  src/OU_GOD2.cpp
  src/OUNITAAT.cpp
  src/OFIRMAI.cpp
  src/OGODRES.cpp
  src/OLIGHTN.cpp
  src/OR_SPY.cpp
  src/OFIRMA.cpp
  src/OTORNADO.cpp
  src/ONEWSSPA.cpp
  src/OF_FACT.cpp
  src/ORAIN3.cpp
  src/OVIDEO.cpp
  src/OMATRIX.cpp
  src/OWEATHER.cpp
  src/OWORLD_Z.cpp
  src/OAI_MAR3.cpp
  src/surface_sdl.cpp
  src/OGRPSEL.cpp
  src/OLIGHTN2.cpp
  src/OGFILE3.cpp
  src/OSNOWRES.cpp
  src/OU_MARI.cpp
  src/file_input_stream.cpp
  src/OFIRMIF3.cpp
  src/OGAMEMP.cpp
  src/ODYNARR.cpp
  src/OAI_DIPL.cpp
  src/OAI_CAP2.cpp
  src/OU_MONS.cpp
  src/OFIRMDRW.cpp
  src/OAI_SPY.cpp
  src/ONEWSFRE.cpp
  src/OAI_CAPT.cpp
  src/ODYNARRB.cpp
  src/OPOWER.cpp
  src/OTALKSPA.cpp
  src/OSLIDCUS.cpp
  src/OFIRM.cpp
  src/OR_TRADE.cpp
  src/OAI_ACT.cpp
  src/OF_MINE2.cpp
  src/ORESX.cpp
  src/OSITEDRW.cpp
  src/OW_WALL.cpp
  src/OFIRM2.cpp
  src/OSNOW1.cpp
  src/OVBROWIF.cpp
  src/OAI_DEFE.cpp
  src/AM.cpp
  src/OROCK.cpp
  src/OFIRMIF2.cpp
  src/OSNOWG.cpp
  src/OSPREOFF.cpp
  src/OSPRTRES.cpp
  src/OF_INN.cpp
  src/OSE.cpp
  src/OF_BASE2.cpp
  src/misc_uuid.cpp
  src/OF_RESE.cpp
  src/OSPATH.cpp
  src/OTALKMSG.cpp
  src/OUNITAT.cpp
  src/OREGION.cpp
  src/OBATTLE.cpp
  src/OAI_SEEK.cpp
  src/OAI_MARI.cpp
  src/OR_TECH.cpp
  src/OGENMAP.cpp
  src/OSKILL.cpp
  src/OUNITAAC.cpp
  src/OGAMMENU.cpp
  src/ODIR.cpp
  src/OGFILE2.cpp
  src/OSYS.cpp
  src/OSPREDBG.cpp
  src/OR_TOWN.cpp
  src/OAI_MILI.cpp
  src/OVGABUF2.cpp
  src/OAI_UNIT.cpp
  src/OSPYA.cpp
  src/OUNITDRW.cpp
  src/ORES.cpp
  src/OUNITRES.cpp
  src/OTOWNBLD.cpp
  src/OTOWNIF.cpp
  src/OWALLRES.cpp
  src/OF_RESE2.cpp
  src/OU_MARI2.cpp
  src/OU_MARIT.cpp
  src/OERRCTRL.cpp
  src/OR_NEWS.cpp
  src/OAI_ECO.cpp
  src/OFILETXT.cpp
  src/player_desc.cpp
  src/OIMGRES.cpp
  src/openal/wav_stream.cpp
  src/openal/openal_audio.cpp
  src/OMISC.cpp
  src/OHILLRES.cpp
  src/OBOX.cpp
  src/ONATIONB.cpp
  src/dbglog.cpp
  src/OGETA.cpp
  src/OMUSIC.cpp
  src/OUNITHB.cpp
  src/OTERRAIN.cpp
  src/OCOLTBL.cpp
  src/OMOUSECR.cpp
  src/OLZW.cpp
  src/OGF_V1.cpp
  src/OSPY2.cpp
  src/OUNITIND.cpp
  src/OTOWNIND.cpp
  src/OU_VEHI.cpp
  src/OF_CAMP2.cpp
  src/OAI_ACT2.cpp
  src/vga_sdl.cpp
  src/OFIRMRES.cpp
  src/file_util.cpp
  src/OGAMCRED.cpp
  src/OSTR.cpp
  src/ONEWS.cpp
  src/OSCROLL.cpp
  src/OSPRITE.cpp
  src/OUNITIF.cpp
  src/OGENHILL.cpp
  src/OSPATHBT.cpp
  src/OAI_MAR2.cpp
  src/OBLOB.cpp
  src/OREBEL.cpp
  src/OUNITA.cpp
  src/OBUTTON.cpp
  src/OBUTT3D.cpp
  src/ODB.cpp
  src/OPLANT.cpp
  src/OREMOTE2.cpp
  src/OMP_CRC.cpp
  src/OFILE.cpp
  src/OTOWNAI.cpp
  src/OMONSRES.cpp
  src/OFLAME.cpp
  src/OAI_MAIN.cpp
  src/OEFFECT.cpp
  src/OGAMHALL.cpp
  src/ONEWS2.cpp
  src/OTOWNRES.cpp
  src/OF_HARB.cpp
  src/OR_NAT.cpp
  src/OHELP.cpp
  src/OSERES.cpp
  src/OF_INN2.cpp
  src/OF_CAMP.cpp
  src/imgfun/generic/I_BAR.cpp
  src/imgfun/generic/IB_ATDM.cpp
  src/imgfun/generic/IB_TD.cpp
  src/imgfun/generic/IB_DW.cpp
  src/imgfun/generic/IB_A.cpp
  src/imgfun/generic/CRC.cpp
  src/imgfun/generic/IB_ATRDM.cpp
  src/imgfun/generic/IC.cpp
  src/imgfun/generic/IB_ATR.cpp
  src/imgfun/generic/IB2.cpp
  src/imgfun/generic/IJ_T.cpp
  src/imgfun/generic/IR.cpp
  src/imgfun/generic/IB_32.cpp
  src/imgfun/generic/I_EREMAP.cpp
  src/imgfun/generic/I_FONT.cpp
  src/imgfun/generic/IR_BAR.cpp
  src/imgfun/generic/IC_R.cpp
  src/imgfun/generic/I_READ.cpp
  src/imgfun/generic/IB_TDM.cpp
  src/imgfun/generic/IR_A.cpp
  src/imgfun/generic/IB_RD.cpp
  src/imgfun/generic/IB_ATD.cpp
  src/imgfun/generic/I_LINE.cpp
  src/imgfun/generic/IB_T.cpp
  src/imgfun/generic/I_SNOW.cpp
  src/imgfun/generic/IB_R.cpp
  src/imgfun/generic/IB_TRD.cpp
  src/imgfun/generic/I_FREMAP.cpp
  src/imgfun/generic/IB_AR.cpp
  src/imgfun/generic/I_PIXEL.cpp
  src/imgfun/generic/IB_AT.cpp
  src/imgfun/generic/IR_AM.cpp
  src/imgfun/generic/IB_ATRD.cpp
  src/imgfun/generic/I_EMASK.cpp
  src/imgfun/generic/IB_TR.cpp
  src/imgfun/generic/I_BLACK.cpp
  src/imgfun/generic/IB_TRDM.cpp
  src/imgfun/generic/IB.cpp
  src/imgfun/generic/IR_M.cpp
  src/OUNITAT2.cpp
  src/OU_CARA2.cpp
  src/OVGABUF.cpp
  src/OGAME.cpp
  src/OB_FLAME.cpp
  src/mem_input_stream.cpp
  src/OGAMSCEN.cpp
  src/OUNITAMT.cpp
  src/OF_MONS.cpp
  src/OU_CARAT.cpp
  src/OGAMEND.cpp
  src/ORAIN2.cpp
  src/OGAMSING.cpp
  src/OVGALOCK.cpp
  src/OF_BASE.cpp
  src/OSPY.cpp
  src/OTALKGER.cpp
  src/file_writer.cpp
  src/OFONT.cpp
  src/OSPRITE2.cpp
  src/OR_RANK.cpp
  src/output_stream.cpp
  src/OAI_BUIL.cpp
  src/OU_CART.cpp
  src/OCRC_STO.cpp
)



add_definitions(-include "${SEVENKAA_COMFIG_INCLUDE_FILE}")
#add_definitions(-D_REENTRANT)
#add_definitions(-DLOCALEDIR="${CMAKE_INSTALL_PREFIX}/share/7kaa")

include_directories("${SEVENKAA_INCLUDE_DIR}")
include_directories(${SDL2_INCLUDE_DIRS})
include_directories("${OPENAL_INCLUDE_DIR}")

add_executable(7kaa
	${H_FILES}
	${CPP_FILES}
	${SEVENKAA_COMFIG_INCLUDE_FILE}
)

target_link_libraries(7kaa ${SDL2_LIBRARIES} "${OPENAL_LIBRARY}" "${SEVENKAA_ENET_LIBRARY}")
