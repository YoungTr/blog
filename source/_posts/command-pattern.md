---
title: 命令模式
date: 2017-09-21 19:40:09
tags: [设计模式]
categories: [设计模式]
---

### 1. 模式动机

在软件设计中，我们经常需要向某些对象发送请求，但是并不知道请求的接收者是谁，也不知道被请求的操作是哪个，我们只需在程序运行时指定具体的请求接收者即可，此时，可以使用命令模式来进行设计，使得请求发送者与请求接收者消除彼此之间的耦合，让对象之间的调用关系更加灵活。

利用命令模式，把请求（例如打开电灯）封装成一个特定的对象（例如客厅电灯对象）。所以，如果对每个按钮都存储一个命令对象，那么当按钮被按下的时候，就可以请命令对象做相应的操作。遥控器并不需要知道工作内容是什么，只要有个命令对象能和正确的对象沟通，把事情做好就OK了。


### 2. 定义命令模式

**命令模式：**将“请求”封装成对象，以便使用不同的请求、队列或者日志来参数化其他对象。命令模式也支持可撤销的操作。

### 3. 模式结构

命令模式包含如下角色：

* Command: 抽象命令类
* ConcreteCommand: 具体命令类
* Invoker: 调用者
* Receiver: 接收者
* Client:客户类

![UML图](/images/command_pattern_uml.png)

### 4.1 代码分析

* Command

```
	public interface Command {
	    void execute();
	}
```

* ConcreteCommand

```
public class LightOnCommand implements Command {

    Light light;

    public LightOnCommand(Light light) {
        this.light = light;
    }

    @Override
    public void execute() {
        light.on();
    }
}
```

* Reciver

```
public class SlotLight implements Light {


    @Override
    public void on() {
        Log.d(LOG_TAG, "on: ");
    }

    @Override
    public void off() {
        Log.d(LOG_TAG, "off: ");
    }
}
```

* Invoker

```
public class RemoteControl {

    private static final int COMMAND_TOTAL_COUNT = 7;

    Command[] onCommands;
    Command[] offCommands;    //这里遥控器需要处理7个开与关的命令，使用相应数组记录这些命令

    public RemoteControl() {

        // 实例初始化这两个这两个开与关的数组
        onCommands = new Command[COMMAND_TOTAL_COUNT];
        offCommands = new Command[COMMAND_TOTAL_COUNT];

        Command noCommand = new NoCommand();
        //将每个操作默认设置为 NoCommand,就不必每次都检验是否都加载了命令
        for (int i = 0; i < COMMAND_TOTAL_COUNT; i++) {
            onCommands[i] = noCommand;
            offCommands[i] = noCommand;
        }
    }

    public void setCommand(int slot, Command onCommand, Command offCommand) {
        onCommands[slot] = onCommand;
        offCommands[slot] = offCommand;
    }

    public void onButtonWasPushed(int slot) {
        onCommands[slot].execute();
    }

    public void offButtonWasPushed(int slot) {
        offCommands[slot].execute();
    }

    public String toString() {
        StringBuffer stringBuff = new StringBuffer();
        stringBuff.append("\n------ Remote Control -------\n");
        for (int i = 0; i < onCommands.length; i++) {
            stringBuff.append("[slot " + i + "] " + onCommands[i].getClass().getName()
                    + "    " + offCommands[i].getClass().getName() + "\n");
        }
        return stringBuff.toString();
    }
}
```

* Client

```
public class RemoteLoader {
 
	public static void main(String[] args) {
		RemoteControl remoteControl = new RemoteControl();
 
		Light livingRoomLight = new Light("Living Room");
		Light kitchenLight = new Light("Kitchen");
		CeilingFan ceilingFan= new CeilingFan("Living Room");
		GarageDoor garageDoor = new GarageDoor("");
		Stereo stereo = new Stereo("Living Room");
  
		LightOnCommand livingRoomLightOn = 
				new LightOnCommand(livingRoomLight);
		LightOffCommand livingRoomLightOff = 
				new LightOffCommand(livingRoomLight);
		LightOnCommand kitchenLightOn = 
				new LightOnCommand(kitchenLight);
		LightOffCommand kitchenLightOff = 
				new LightOffCommand(kitchenLight);
  
		CeilingFanOnCommand ceilingFanOn = 
				new CeilingFanOnCommand(ceilingFan);
		CeilingFanOffCommand ceilingFanOff = 
				new CeilingFanOffCommand(ceilingFan);
 
		GarageDoorUpCommand garageDoorUp =
				new GarageDoorUpCommand(garageDoor);
		GarageDoorDownCommand garageDoorDown =
				new GarageDoorDownCommand(garageDoor);
 
		StereoOnWithCDCommand stereoOnWithCD =
				new StereoOnWithCDCommand(stereo);
		StereoOffCommand  stereoOff =
				new StereoOffCommand(stereo);
 
		remoteControl.setCommand(0, livingRoomLightOn, livingRoomLightOff);
		remoteControl.setCommand(1, kitchenLightOn, kitchenLightOff);
		remoteControl.setCommand(2, ceilingFanOn, ceilingFanOff);
		remoteControl.setCommand(3, stereoOnWithCD, stereoOff);
  
		System.out.println(remoteControl);
 
		remoteControl.onButtonWasPushed(0);
		remoteControl.offButtonWasPushed(0);
		remoteControl.onButtonWasPushed(1);
		remoteControl.offButtonWasPushed(1);
		remoteControl.onButtonWasPushed(2);
		remoteControl.offButtonWasPushed(2);
		remoteControl.onButtonWasPushed(3);
		remoteControl.offButtonWasPushed(3);
	}
}
```

* 输出

```
  ------ Remote Control -------		
  [slot 0] com.youngtr.factorypattern.remote.LightOnCommand    com.youngtr.factorypattern.remote.LightOffCommand
  [slot 1] com.youngtr.factorypattern.remote.LightOnCommand    com.youngtr.factorypattern.remote.LightOffCommand
  [slot 2] com.youngtr.factorypattern.remote.CeilingFanOnCommand    com.youngtr.factorypattern.remote.CeilingFanOffCommand
  [slot 3] com.youngtr.factorypattern.remote.StereoOnWithCDCommand    com.youngtr.factorypattern.remote.StereoOffCommand
  [slot 4] com.youngtr.factorypattern.remote.NoCommand    com.youngtr.factorypattern.remote.NoCommand
  [slot 5] com.youngtr.factorypattern.remote.NoCommand    com.youngtr.factorypattern.remote.NoCommand
  [slot 6] com.youngtr.factorypattern.remote.NoCommand    com.youngtr.factorypattern.remote.NoCommand
  Living Room light is on
  Living Room light is off
  Kitchen light is on
  Kitchen light is off
  Living Room ceiling fan is on high
  Living Room ceiling fan is off
  Living Room stereo is on
  Living Room stereo is set for CD input
  Living Room Stereo volume set to 11
  Living Room stereo is off

```

### 4.2 支持撤销

当命令支持撤销时，该命令就必须提供和 execute() 方法相反的 undo() 方法。

在 Command 接口中加入 undo() 方法

```
public interface Command {
	 void execute();
	
	void undo();
}
```

使用状态实现撤销

```
public class CeilingFan {
	String location = "";
	int level;
	public static final int HIGH = 2;
	public static final int MEDIUM = 1;
	public static final int LOW = 0;
 
	public CeilingFan(String location) {
		this.location = location;
	}
  
	public void high() {
		// turns the ceiling fan on to high
		level = HIGH;
		System.out.println(location + " ceiling fan is on high");
 
	} 

	public void medium() {
		// turns the ceiling fan on to medium
		level = MEDIUM;
		System.out.println(location + " ceiling fan is on medium");
	}

	public void low() {
		// turns the ceiling fan on to low
		level = LOW;
		System.out.println(location + " ceiling fan is on low");
	}
 
	public void off() {
		// turns the ceiling fan off
		level = 0;
		System.out.println(location + " ceiling fan is off");
	}
 
	public int getSpeed() {
		return level;
	}
}
```

加入撤销到吊扇的命令类，每次执行execute()方法，将 execute() 被执行前的状态记录下来

```
public class CeilingFanOnCommand implements Command {
    CeilingFan ceilingFan;
    int prevSpeed;

    public CeilingFanOnCommand(CeilingFan ceilingFan) {
        this.ceilingFan = ceilingFan;
    }

    public void execute() {
        prevSpeed = ceilingFan.getSpeed();
        ceilingFan.high();
    }

    @Override
    public void undo() {
        if (prevSpeed == CeilingFan.MEDIUM) {
            ceilingFan.medium();
        } else if (prevSpeed == CeilingFan.LOW) {
            ceilingFan.low();
        }
    }
}
```

### 5. 总结

1. 命令模式将发出请求的对象和执行请求的对象解耦。
2. 在被解耦的两者之间是通过命令对象进行沟通的，命令对象封装了接收者一个或一组动作。
3. 调用者通过调用命令对象的execute()发出请求，这会使得接收者的动作被调用。
4. 调用者可以接收命令当作参数，甚至在运行时动态地进行。
5. 命令支持撤销，做法是实现一个 undo() 方法来回到 execute() 被执行前的状态。



