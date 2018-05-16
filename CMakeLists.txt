cmake_minimum_required(VERSION 2.8)
project(memory_gen)

include(ChipCMake)

# Create an IP entry without source file
specify_design()

function(add_memory_macro _MEM_NAME)
  # The function will be called out of this IP (e.g. the PARENT_SCOPE)
  list(APPEND CMAKE_MODULE_PATH "${memory_gen_DIR}/cmake")
  find_package(TSMC90Memory REQUIRED)

  set(mem_spec ${CMAKE_CURRENT_BINARY_DIR}/${_MEM_NAME}.spec)
  set(sim_model ${CMAKE_CURRENT_BINARY_DIR}/${_MEM_NAME}.v)
  set(sim_wrapper ${CMAKE_CURRENT_BINARY_DIR}/${_MEM_NAME}_wrapper.v)

  # Cannot use specify_design() here, since the function will be called out of this IP
  add_ip_sources(memory_gen ${sim_model} ${sim_wrapper})

  set(options SRAM RF)
  set(oneValueArgs BIT_WIDTH FREQ MUX SIZE)
  set(multiValueArgs "")
  cmake_parse_arguments(MEMGEN "${options}" "${oneValueArgs}" "${multiValueArgs}" ${ARGN})
  set(MEMGEN_INSTNAME ${_MEM_NAME})

  if(${MEMGEN_RF})
    add_custom_command(OUTPUT ${sim_model}
      COMMAND ${TSMC90Memory_SRAM_1P_BIN} verilog -spec ${mem_spec}
      COMMENT "Generating memory"
    )

    configure_file(${memory_gen_DIR}/src/regfile_1p.spec.in
      ${CMAKE_CURRENT_BINARY_DIR}/${_MEM_NAME}.spec)
  else() # SRAM
    add_custom_command(OUTPUT ${sim_model}
      COMMAND ${TSMC90Memory_SRAM_1P_BIN} verilog -spec ${mem_spec}
      COMMENT "Generating memory"
    )

    configure_file(${memory_gen_DIR}/src/wrapper.sv.in ${sim_wrapper})
    configure_file(${memory_gen_DIR}/src/sram_1p.spec.in ${mem_spec})
  endif()
  add_custom_target(memory_gen DEPENDS ${sim_model})
  add_dependencies(c2mk_pre_simulation memory_gen)
endfunction()
