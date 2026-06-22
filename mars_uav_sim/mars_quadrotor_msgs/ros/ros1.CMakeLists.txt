#------------------------------------------------------------------
cmake_minimum_required(VERSION 2.8.3)
project(quadrotor_msgs)

find_package(catkin REQUIRED COMPONENTS
        #armadillo
        roscpp
        nav_msgs
        geometry_msgs
        std_msgs
        message_generation
)

set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)

include_directories(
        ${catkin_INCLUDE_DIRS}
        include
)

#
add_message_files(
        FILES
        PositionCommand.msg
        MpcPositionCommand.msg
        PolynomialTrajectory.msg
        Px4ctrlDebug.msg
        TrakingPerformance.msg
        QuadrotorState.msg
        TakeoffLand.msg
)

generate_messages(
        DEPENDENCIES
        geometry_msgs
        nav_msgs
        std_msgs
)

catkin_package(
        #INCLUDE_DIRS include
        LIBRARIES
        #CATKIN_DEPENDS geometry_msgs nav_msgs
        #DEPENDS system_lib
        CATKIN_DEPENDS message_runtime
)