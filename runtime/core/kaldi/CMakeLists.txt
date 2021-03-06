cmake_minimum_required(VERSION 3.10 FATAL_ERROR)

project(kaldi)

add_library(kaldi-base
base/kaldi-error.cc
base/kaldi-math.cc
)
target_link_libraries(kaldi-base PUBLIC utils)

add_library(kaldi-util
util/kaldi-io.cc
util/parse-options.cc
util/simple-io-funcs.cc
util/text-utils.cc
)
target_link_libraries(kaldi-util PUBLIC kaldi-base)

add_library(kaldi-lat
lat/determinize-lattice-pruned.cc
lat/lattice-functions.cc
)
target_link_libraries(kaldi-lat PUBLIC kaldi-util)

add_library(kaldi-decoder
decoder/lattice-faster-decoder.cc
decoder/lattice-faster-online-decoder.cc
)
target_link_libraries(kaldi-decoder PUBLIC kaldi-lat)

add_library(kaldi-lm
lm/arpa-file-parser.cc
lm/arpa-lm-compiler.cc
)
target_link_libraries(kaldi-lm PUBLIC kaldi-util)

add_library(kaldi-fstext
fstext/kaldi-fst-io.cc
)
target_link_libraries(kaldi-fstext PUBLIC kaldi-util)

# Arpa binary
add_executable(arpa2fst lmbin/arpa2fst.cc)
target_link_libraries(arpa2fst PUBLIC kaldi-lm)

# FST tools binary
set(FST_BINS
fstaddselfloops
fstdeterminizestar
fstisstochastic
fstminimizeencoded
fsttablecompose
)

foreach(name IN LISTS FST_BINS)
  add_executable(${name} fstbin/${name}.cc)
  # dl is for dynamic linking, otherwise there is a linking error on linux
  target_link_libraries(${name} PUBLIC kaldi-fstext dl)
endforeach()
