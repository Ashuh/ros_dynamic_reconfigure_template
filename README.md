# ros_dynamic_reconfigure_template
NOTE: Replace `package_name` and `cfg_file_name` with their actual names 
## Step 1: Create a cfg File
Create a cfg file in package_name/cfg and follow the template below

### The cfg File Explained
This first lines are pretty simple, they just initialize ros and import the parameter generator.
```
#!/usr/bin/env python
PACKAGE = "package_name"

from dynamic_reconfigure.parameter_generator_catkin import *
gen = ParameterGenerator()
```
Now that we have a generator we can start to define parameters. The add function adds a parameter to the list of parameters. It takes a few different arguments:
- name - a string which specifies the name under which this parameter should be stored
- paramtype - defines the type of value stored, and can be any of int_t, double_t, str_t, or bool_t
- level - A bitmask which will later be passed to the dynamic reconfigure callback. When the callback is called all of the level values for parameters that have been changed are ORed together and the resulting value is passed to the callback.
- description - string which describes the parameter
- default - specifies the default value
- min - specifies the min value (optional and does not apply to strings and bools)
- max - specifies the max value (optional and does not apply to strings and bools)\

These lines simply define more parameters of the different types.
```
gen.add("int_param",    int_t,    0, "An Integer parameter", 50,  0, 100)
gen.add("double_param", double_t, 0, "A double parameter",    .5, 0,   1)
gen.add("str_param",    str_t,    0, "A string parameter",  "Hello World")
gen.add("bool_param",   bool_t,   0, "A Boolean parameter",  True)
```
Here we define an integer whose value is set by an enum. To do this we call gen.enum and pass it a list of constants followed by a description of the enum. Now that we have created an enum we can now pass it to the generator. Now the param can be set with "Small" or "Medium" rather than 0 or 1.
```
size_enum = gen.enum([ gen.const("Small",      int_t, 0, "A small constant"),
                       gen.const("Medium",     int_t, 1, "A medium constant"),
                       gen.const("Large",      int_t, 2, "A large constant"),
                       gen.const("ExtraLarge", int_t, 3, "An extra large constant")],
                     "An enum to set size")

gen.add("size", int_t, 0, "A size parameter which is edited via an enum", 1, 0, 3, edit_method=size_enum)
```
The last line simply tells the generator to generate the necessary files and exit the program. The second parameter is the name of a node this could run in (used to generate documentation only), the third parameter is a name prefix the generated files will get (e.g. "<name>Config.h" for c++, or "<name>Config.py" for python

NOTE: The third parameter should be equal to the cfg file name, without extension. Otherwise the libraries will be generated in every build, forcing a recompilation of the nodes which use them.
```
exit(gen.generate(PACKAGE, "package_name", "cfg_file_name"))
```
## Step 2: Make the cfg File executable
In order to make this cfg file usable it must be executable, so lets use the following command to make it excecutable
```
chmod a+x cfg/cfg_file_name.cfg
```
## Step 3: Setup CMakeLists.txt and package.xml
Add the following lines to our CMakeLists.txt.
```
find_package(catkin REQUIRED COMPONENTS
  dynamic_reconfigure
)

#find_package(catkin REQUIRED dynamic_reconfigure)
generate_dynamic_reconfigure_options(
  cfg/cfg_file_name.cfg
)
```
Add the following lines to our package.xml.
```
<build_depend>dynamic_reconfigure</build_depend>
<build_export_depend>dynamic_reconfigure</build_export_depend>
<exec_depend>dynamic_reconfigure</exec_depend>
```
## Step 4: Setup Dynamic Reconfigure for a ROS Node
Add the following lines of code into your ROS node

### The Breakdown
Here we just include the necessary header files for our node. Take note of package_name/<cfg_file_name>Config.h this is the header file generated by dynamic_reconfigure from our config file.
```
#include <ros/ros.h>

#include <dynamic_reconfigure/server.h>
#include <package_name/<cfg_file_name>Config.h>>
```
This is the callback that will get called when the dynamic_reconfigure server is sent a new configuration. It takes two parameters, the first is the new config. The second is the level, which is the result of ORing together all of level values of the parameters that have changed. What you want to do with the level value is entirely up to you, for now it is unnecessary. All we are going to do in the callback is print out the configuration.
```
void callback(package_name::<cfg_file_name>Config &config, uint32_t level) {
  ROS_INFO("Reconfigure Request: %d %f %s %s %d", 
            config.int_param, config.double_param, 
            config.str_param.c_str(), 
            config.bool_param?"True":"False", 
            config.size);
}
```
In our main function we simply initialize our node and define the dynamic_reconfigure server, passing it our config type. As long as the server lives (in this case until the end of main()), the node listens to reconfigure requests.
```
int main(int argc, char **argv) {
  ros::init(argc, argv, "node_name");

  dynamic_reconfigure::Server<package_name::<cfg_file_name>Config> server;
```
Next we define a variable to represent our callback and then send it to the server. Now when the server gets a reconfiguration request it will call our callback function.

NOTE:If the callback if a member function of class use f = boost::bind(&callback, x, _1, _2) instead, where x is an instance of the class (or this if called from inside the class).
```
dynamic_reconfigure::Server<package_name::<cfg_file_name>Config>::CallbackType f;

  f = boost::bind(&callback, _1, _2);
  server.setCallback(f);
```
Lastly we spin the node.
```
  ROS_INFO("Spinning node");
  ros::spin();
  return 0;
}
```
## Run the GUI
```
rosrun rqt_reconfigure rqt_reconfigure
```

## References
- http://wiki.ros.org/dynamic_reconfigure/Tutorials/HowToWriteYourFirstCfgFile
- http://wiki.ros.org/dynamic_reconfigure/Tutorials/SettingUpDynamicReconfigureForANode%28cpp%29
