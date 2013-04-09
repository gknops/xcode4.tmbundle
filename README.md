# Introduction

`Xcode4.tmbundle` provides simple build and run support for Xcode4 projects. It requires [`ApLo.tmbundle`](https://github.com/gknops/aplo.tmbundle).

This bundle can also run iOS projects in the simulator.

Note that you still need to use Xcode to add and remove files to/from the project itself, edit `.xib` files, for debugging, unit testing etc. But this bundle will allow you to use TextMate for editing, compilation and application execution. 

![BuldAndRun](https://github.com/gknops/Xcode4.tmbundle/raw/master/BuldAndRun.png)


# Installation

Open a terminal window and enter these commands:

	cd ~/Library/Application\ Support/Avian/Bundles
	git clone --recursive git://github.com/gknops/xcode4.tmbundle.git


# Use

While in an Objective-C source, use `⌘r` to build and run the current project, or `⇧⌘X` to access other commands (`Build`, `Run`, `Help`).

**NOTE**: The bundle does not parse the `.xcodeproj` file (which is quite complex and a bit of a moving target). So in determines the path to the executable by parsing the output of the build command, and stores the path to the executable in a file named `.aploExecutablePathFor<TARGET>` in the project root. So the run command will fail until the project was compiled at least once before using the bundle commands, and at least one file must have had changes.

# Clickable log output

The parser parsing the application output will look for certain patterns and convert them into clickable links that get you right to the source file. This is a very useful debug tool. It can also highlight warning and error output.

In the simplest case you could just include this:

	#define	DLog(format,...) NSLog(@"%s:%d: "format,__FILE__,__LINE__,##__VA_ARGS__);

and start using `DLog` instead of `NSLog`.

Here is a more complete boilerplate I usually include in my projects:


	//*****************************************************************************
	// Macros
	//*****************************************************************************
	#ifdef __clang__
		#define _CMD NSStringFromSelector(_cmd)
	#else
		#define _CMD [NSString stringWithFormat:@"%s",_cmd]
	#endif
	
	#define IS_MAIN_QUEUE (dispatch_get_current_queue()==dispatch_get_main_queue())
	
	//*****************************************************************************
	// Logging
	//*****************************************************************************
	#ifdef DEBUG
		#define	DLog(format,...) NSLog(@"%s:%d: "format,__FILE__,__LINE__,##__VA_ARGS__);
	#else
		#define DLog(...)
	#endif
	
	#define	WLog(format,...) NSLog(@"%s:%d: WARNING: "format,__FILE__,__LINE__,##__VA_ARGS__);
	#define	ERRLog(format,...) NSLog(@"%s:%d: ERROR: "format,__FILE__,__LINE__,##__VA_ARGS__);
	#define	ASSERTLog(format,...) NSLog(@"%s:%d: ASSERTION: "format,__FILE__,__LINE__,##__VA_ARGS__);
	
	//*****************************************************************************
	// Assertions
	//*****************************************************************************
	#define ASSERT(cond,format,...)	if(!(cond)) { ASSERTLog(format,##__VA_ARGS__); NSAssert(NO,@""); }
	#define SUBCLASS_RESPONSIBILITY ASSERT(NO,@"%@ failed to implement %@!",NSStringFromClass([self class]),_CMD);
	

So now all `DLog` will be automatically removed in non-debug builds. `WLog` and `ERRLog` output is highlighted. And `ASSERT` replaces the annoying `NSAssert1()`, `NSAssert2()` etc calls.

The ApLo output will key off prefixes like `WARNING:` and `ERROR:` as seen above and color the output. These prefixes and colors are supported:

INFO
:	green
WARNING
:	GoldenRod
ERROR
:	FireBrick
ASSERTION
:	white on FireBrick background

You can also define custom colors, using a line like:

	NSLog(@"ApLoColorDefine A_PREFIX HTML_COLOR");

where `A_PREFIX` is the prefix you will be using, and `HTML_COLOR` is any valid HTML color designation (name, hex color etc) that can be used in a style.


# Multiple build targets

If your project has multiple build targets, you can use `.tm_properties` files in various directories to control which target will be build. Just set the `APLO_WINDOW_NAME` variable to the name of the desired target.

If you have source shared between multiple targets, use the `.tm_properties` file in the project root to set the `APLO_DEFAULT_TO_LAST` variable to `yes`. This will cause the bundle to memorize the last built target in a file named `.aploLastBuild` in the project root directory. 


# iOS projects

This bundle can build iOS projects and run them in the simulator, using [ios-sim][] which is included in the bundle.

To build and run an iOS project some extra variables are required in the projects root `.tm_properties` file. Here an example:

	APLO_EXTRA_XCODE_ARGS	= "-sdk iphonesimulator4.3 -arch i386"
	APLO_IOSSIM_ARGS		= '--sdk 4.3 --family ipad'


# .tm_properties variables

## APLO\_WINDOW\_NAME

Sets the window name for ApLo. It not set it will default to the last path component of the `TM_PROJECT_DIRECTORY` environment variable.

## APLO\_DEFAULT\_TO\_LAST

Remember the last built scheme. See the [Multiple build targets]() section for more info.

## APLO\_XCODE\_SCHEME

Set the Xcode scheme to be used for building. Will default to `APLO_WINDOW_NAME`.

## APLO\_EXTRA\_XCODE\_ARGS

Additional arguments added to the `xcodebuild` command line.

## APLO\_IOSSIM\_ARGS

When defined (even if empty) `ios-sim` is used to run the built application in the IOS simulator.

[ios-sim]:	https://github.com/Fingertips/ios-sim
