MACRO(CHECK_DEFINED VAR)
    IF (NOT DEFINED ${VAR})
        MESSAGE(FATAL_ERROR "NOT DEFINED ${VAR}")
    ENDIF()
    MESSAGE("${VAR}: ${${VAR}}")
ENDMACRO()

CHECK_DEFINED(PHXPF_SERVER_WORKDIR)
CHECK_DEFINED(PHXPF_SERVER_BINARY)
CHECK_DEFINED(PHXPF_SERVER_TOOL)
CHECK_DEFINED(PHXPF_SERVER_NODENUM)
CHECK_DEFINED(PHXPF_SERVER_ADDR)

CHECK_DEFINED(PF_SERVER_WORKDIR)
CHECK_DEFINED(PF_SERVER_BINARY)
CHECK_DEFINED(PF_SERVER_TOOL)
CHECK_DEFINED(PF_SERVER_NODENUM)
CHECK_DEFINED(PF_SERVER_ADDR)

CHECK_DEFINED(SAMPLE_CMAKE_FILE)
CHECK_DEFINED(SERVER_START_SCRIPT)
CHECK_DEFINED(SERVER_STOP_SCRIPT)

MACRO(ADD_SAMPLE_CUSTOM_TEST_TARGET NAME SERVER_WORKDIR SERVER_BINARY SERVER_TOOL SERVER_NODENUM SERVER_ADDR SAMPLE_CMAKE_FILE  SERVER_START_SCRIPT SERVER_STOP_SCRIPT)
    ADD_CUSTOM_TARGET(${NAME}
            COMMAND ${CMAKE_COMMAND} -DSERVER_WORKDIR=${SERVER_WORKDIR} -DSERVER_NODENUM=${SERVER_NODENUM} -P ${SAMPLE_CMAKE_FILE}
            COMMAND sh ${SERVER_START_SCRIPT} ${SERVER_BINARY} ${SERVER_WORKDIR}/sample ${SERVER_NODENUM}
            COMMAND ${SERVER_TOOL} -s ${SERVER_ADDR} -f filter -p 18555476 -v "123,456,789"
            COMMAND ${SERVER_TOOL} -s ${SERVER_ADDR} -f update -p 18555476 -v "123,456,789"
            COMMAND ${SERVER_TOOL} -s ${SERVER_ADDR} -f delete -p 18555476 -v "123,456,789"
            COMMAND ${SERVER_TOOL} -s ${SERVER_ADDR} -f filterone -p 18555476 -v "123,456,789"
            COMMAND ${SERVER_TOOL} -s ${SERVER_ADDR} -f updateone -p 18555476 -v "123,456,789"
            COMMAND ${SERVER_TOOL} -s ${SERVER_ADDR} -f deleteone -p 18555476 -v "123,456,789"
            COMMAND sh ${SERVER_STOP_SCRIPT} ${SERVER_BINARY} ${SERVER_WORKDIR}/sample ${SERVER_NODENUM})
ENDMACRO()

MACRO(ADD_SAMPLE_CUSTOM_PYTEST_TARGET NAME SERVER_WORKDIR SERVER_BINARY SERVER_TOOL SERVER_NODENUM SERVER_ADDR SAMPLE_CMAKE_FILE  SERVER_START_SCRIPT SERVER_STOP_SCRIPT TEST_FILE)
    ADD_CUSTOM_TARGET(${NAME}
            COMMAND ${CMAKE_COMMAND} -DSERVER_WORKDIR=${SERVER_WORKDIR} -DSERVER_NODENUM=${SERVER_NODENUM} -P ${SAMPLE_CMAKE_FILE}
            COMMAND sh ${SERVER_START_SCRIPT} ${SERVER_BINARY} ${SERVER_WORKDIR}/sample ${SERVER_NODENUM}

            COMMAND PYTHONPATH=${PROJECT_BINARY_DIR}/python:$ENV{PYTHONPATH} python ${TEST_FILE}
            COMMAND sh ${SERVER_STOP_SCRIPT} ${SERVER_BINARY} ${SERVER_WORKDIR}/sample ${SERVER_NODENUM}
            DEPENDS python
            SOURCES ${TEST_TOOLS_FILES} ${TEST_FILE})
ENDMACRO()

MACRO(ADD_SAMPLE_CUSTOM_BENCHMARK_TARGET NAME BENCHMARK_PROCESS_NUM BENCHMARK_THREAD_NUM BENCHMARK_LOOP_NUM SERVER_WORKDIR SERVER_BINARY SERVER_TOOL SERVER_NODENUM SERVER_ADDR SAMPLE_CMAKE_FILE  SERVER_START_SCRIPT SERVER_STOP_SCRIPT TEST_FILE)
    ADD_CUSTOM_TARGET(${NAME}
            COMMAND ${CMAKE_COMMAND} -DSERVER_WORKDIR=${SERVER_WORKDIR} -DSERVER_NODENUM=${SERVER_NODENUM} -P ${SAMPLE_CMAKE_FILE}
            COMMAND sh ${SERVER_START_SCRIPT} ${SERVER_BINARY} ${SERVER_WORKDIR}/sample ${SERVER_NODENUM}

            COMMAND PYTHONPATH=${PROJECT_BINARY_DIR}/python:$ENV{PYTHONPATH} python ${TEST_FILE} ${BENCHMARK_PROCESS_NUM} ${BENCHMARK_THREAD_NUM} ${BENCHMARK_LOOP_NUM}
            COMMAND sh ${SERVER_STOP_SCRIPT} ${SERVER_BINARY} ${SERVER_WORKDIR}/sample ${SERVER_NODENUM}
            DEPENDS python
            SOURCES ${TEST_TOOLS_FILES} ${TEST_FILE})
ENDMACRO()

ADD_SAMPLE_CUSTOM_TEST_TARGET(standalone_sample_test ${PF_SERVER_WORKDIR} ${PF_SERVER_BINARY} ${PF_SERVER_TOOL} ${PF_SERVER_NODENUM} ${PF_SERVER_ADDR} ${SAMPLE_CMAKE_FILE} ${SERVER_START_SCRIPT} ${SERVER_STOP_SCRIPT})
ADD_SAMPLE_CUSTOM_TEST_TARGET(phxpfilter_sample_test ${PHXPF_SERVER_WORKDIR} ${PHXPF_SERVER_BINARY} ${PHXPF_SERVER_TOOL} ${PHXPF_SERVER_NODENUM} ${PHXPF_SERVER_ADDR} ${SAMPLE_CMAKE_FILE} ${SERVER_START_SCRIPT} ${SERVER_STOP_SCRIPT})

ADD_SAMPLE_CUSTOM_PYTEST_TARGET(standalone_pytest ${PF_SERVER_WORKDIR} ${PF_SERVER_BINARY} ${PF_SERVER_TOOL} ${PF_SERVER_NODENUM} ${PF_SERVER_ADDR} ${SAMPLE_CMAKE_FILE} ${SERVER_START_SCRIPT} ${SERVER_STOP_SCRIPT} ${STANDALONE_PYTEST_FILE})
ADD_SAMPLE_CUSTOM_PYTEST_TARGET(phxpfilter_pytest ${PHXPF_SERVER_WORKDIR} ${PHXPF_SERVER_BINARY} ${PHXPF_SERVER_TOOL} ${PHXPF_SERVER_NODENUM} ${PHXPF_SERVER_ADDR} ${SAMPLE_CMAKE_FILE} ${SERVER_START_SCRIPT} ${SERVER_STOP_SCRIPT} ${PHXPFILTER_PYTEST_FILE})

ADD_SAMPLE_CUSTOM_BENCHMARK_TARGET(standalone_benchmark_2_2_50 2 2 50 ${PF_SERVER_WORKDIR} ${PF_SERVER_BINARY} ${PF_SERVER_TOOL} ${PF_SERVER_NODENUM} ${PF_SERVER_ADDR} ${SAMPLE_CMAKE_FILE} ${SERVER_START_SCRIPT} ${SERVER_STOP_SCRIPT} ${STANDALONE_BENCHMARK_FILE})
ADD_SAMPLE_CUSTOM_BENCHMARK_TARGET(phxpfilter_benchmark_2_2_50 2 2 50 ${PHXPF_SERVER_WORKDIR} ${PHXPF_SERVER_BINARY} ${PHXPF_SERVER_TOOL} ${PHXPF_SERVER_NODENUM} ${PHXPF_SERVER_ADDR} ${SAMPLE_CMAKE_FILE} ${SERVER_START_SCRIPT} ${SERVER_STOP_SCRIPT} ${PHXPFILTER_BENCHMARK_FILE})
ADD_SAMPLE_CUSTOM_BENCHMARK_TARGET(standalone_benchmark_4_4_50 4 4 50 ${PF_SERVER_WORKDIR} ${PF_SERVER_BINARY} ${PF_SERVER_TOOL} ${PF_SERVER_NODENUM} ${PF_SERVER_ADDR} ${SAMPLE_CMAKE_FILE} ${SERVER_START_SCRIPT} ${SERVER_STOP_SCRIPT}  ${STANDALONE_BENCHMARK_FILE})
ADD_SAMPLE_CUSTOM_BENCHMARK_TARGET(phxpfilter_benchmark_4_4_50 4 4 50 ${PHXPF_SERVER_WORKDIR} ${PHXPF_SERVER_BINARY} ${PHXPF_SERVER_TOOL} ${PHXPF_SERVER_NODENUM} ${PHXPF_SERVER_ADDR} ${SAMPLE_CMAKE_FILE} ${SERVER_START_SCRIPT} ${SERVER_STOP_SCRIPT} ${PHXPFILTER_BENCHMARK_FILE})
ADD_SAMPLE_CUSTOM_BENCHMARK_TARGET(standalone_benchmark_4_4_500 4 4 500 ${PF_SERVER_WORKDIR} ${PF_SERVER_BINARY} ${PF_SERVER_TOOL} ${PF_SERVER_NODENUM} ${PF_SERVER_ADDR} ${SAMPLE_CMAKE_FILE} ${SERVER_START_SCRIPT} ${SERVER_STOP_SCRIPT} ${STANDALONE_BENCHMARK_FILE})
ADD_SAMPLE_CUSTOM_BENCHMARK_TARGET(phxpfilter_benchmark_4_4_500 4 4 500 ${PHXPF_SERVER_WORKDIR} ${PHXPF_SERVER_BINARY} ${PHXPF_SERVER_TOOL} ${PHXPF_SERVER_NODENUM} ${PHXPF_SERVER_ADDR} ${SAMPLE_CMAKE_FILE} ${SERVER_START_SCRIPT} ${SERVER_STOP_SCRIPT} ${PHXPFILTER_BENCHMARK_FILE})
ADD_SAMPLE_CUSTOM_BENCHMARK_TARGET(phxpfilter_benchmark_4_4_5000 4 4 5000 ${PHXPF_SERVER_WORKDIR} ${PHXPF_SERVER_BINARY} ${PHXPF_SERVER_TOOL} ${PHXPF_SERVER_NODENUM} ${PHXPF_SERVER_ADDR} ${SAMPLE_CMAKE_FILE} ${SERVER_START_SCRIPT} ${SERVER_STOP_SCRIPT} ${PHXPFILTER_BENCHMARK_FILE})
ADD_SAMPLE_CUSTOM_BENCHMARK_TARGET(standalone_benchmark_4_4_5000 4 4 5000 ${PF_SERVER_WORKDIR} ${PF_SERVER_BINARY} ${PF_SERVER_TOOL} ${PF_SERVER_NODENUM} ${PF_SERVER_ADDR} ${SAMPLE_CMAKE_FILE} ${SERVER_START_SCRIPT} ${SERVER_STOP_SCRIPT} ${STANDALONE_BENCHMARK_FILE})
ADD_SAMPLE_CUSTOM_BENCHMARK_TARGET(phxpfilter_benchmark_8_4_5000 8 4 5000 ${PHXPF_SERVER_WORKDIR} ${PHXPF_SERVER_BINARY} ${PHXPF_SERVER_TOOL} ${PHXPF_SERVER_NODENUM} ${PHXPF_SERVER_ADDR} ${SAMPLE_CMAKE_FILE} ${SERVER_START_SCRIPT} ${SERVER_STOP_SCRIPT} ${PHXPFILTER_BENCHMARK_FILE})
ADD_SAMPLE_CUSTOM_BENCHMARK_TARGET(standalone_benchmark_8_4_5000 8 4 5000 ${PF_SERVER_WORKDIR} ${PF_SERVER_BINARY} ${PF_SERVER_TOOL} ${PF_SERVER_NODENUM} ${PF_SERVER_ADDR} ${SAMPLE_CMAKE_FILE} ${SERVER_START_SCRIPT} ${SERVER_STOP_SCRIPT} ${STANDALONE_BENCHMARK_FILE})
ADD_SAMPLE_CUSTOM_BENCHMARK_TARGET(phxpfilter_benchmark_16_4_5000 16 4 5000 ${PHXPF_SERVER_WORKDIR} ${PHXPF_SERVER_BINARY} ${PHXPF_SERVER_TOOL} ${PHXPF_SERVER_NODENUM} ${PHXPF_SERVER_ADDR} ${SAMPLE_CMAKE_FILE} ${SERVER_START_SCRIPT} ${SERVER_STOP_SCRIPT} ${PHXPFILTER_BENCHMARK_FILE})
ADD_SAMPLE_CUSTOM_BENCHMARK_TARGET(standalone_benchmark_16_4_5000 16 4 5000 ${PF_SERVER_WORKDIR} ${PF_SERVER_BINARY} ${PF_SERVER_TOOL} ${PF_SERVER_NODENUM} ${PF_SERVER_ADDR} ${SAMPLE_CMAKE_FILE} ${SERVER_START_SCRIPT} ${SERVER_STOP_SCRIPT} ${STANDALONE_BENCHMARK_FILE})
