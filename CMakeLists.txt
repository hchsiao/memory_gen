cmake_minimum_required(VERSION 2.8)
project(memory_gen)

include(ChipCMake)

# Create an IP entry without source file
get_property(is_specified GLOBAL PROPERTY memory_gen_IP_SOURCES SET)
if(NOT is_specified)
  specify_design()
endif()

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
      COMMAND ${TSMC90Memory_RF_1P_BIN} verilog -spec ${mem_spec}
      COMMENT "Generating memory"
    )

    set(MEMGEN_PHY_SIZE 0)
    while(MEMGEN_PHY_SIZE LESS MEMGEN_SIZE)
      math(EXPR MEMGEN_PHY_SIZE "${MEMGEN_PHY_SIZE}+${MEMGEN_MUX}*2")
    endwhile()
    message("Requested memory size: ${MEMGEN_SIZE}, actual physical size ${MEMGEN_PHY_SIZE}")

    configure_file(${memory_gen_DIR}/src/wrapper.sv.in ${sim_wrapper})
    configure_file(${memory_gen_DIR}/src/regfile_1p.spec.in ${mem_spec})
  else() # SRAM
    add_custom_command(OUTPUT ${sim_model}
      COMMAND ${TSMC90Memory_SRAM_1P_BIN} verilog -spec ${mem_spec}
      COMMENT "Generating memory"
    )

    set(MEMGEN_PHY_SIZE 0)
    while(MEMGEN_PHY_SIZE LESS MEMGEN_SIZE)
      math(EXPR MEMGEN_PHY_SIZE "${MEMGEN_PHY_SIZE}+${MEMGEN_MUX}*4")
    endwhile()
    message("Requested memory size: ${MEMGEN_SIZE}, actual physical size ${MEMGEN_PHY_SIZE}")

    configure_file(${memory_gen_DIR}/src/wrapper.sv.in ${sim_wrapper})
    configure_file(${memory_gen_DIR}/src/sram_1p.spec.in ${mem_spec})
  endif()
  add_custom_target(memory_gen_target_${_MEM_NAME} DEPENDS ${sim_model})
  add_dependencies(c2mk_pre_simulation memory_gen_target_${_MEM_NAME})
endfunction()
