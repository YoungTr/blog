---
title: 外观模式
date: 2017-10-14 10:47:01
tags: [设计模式]
categories: [设计模式]
---

### 1. 模式动机

通过实现一个提供更合理的接口的外观类，可以将一个复杂的子系统变得容易使用。

如果需要复杂子系统的各种功能，还是可以使用原来复杂接口的；但是如果需要的是一个方便的接口，就可以使用外观。


### 2. 模式定义

**外观模式**提供了一个统一的接口，用来访问子系统中的一群接口。外观定义了一个高层接口，让系统更容易使用。

外观模式让客户和子系统之间避免紧耦合。

### 3. 模式分析

#### 3.1 外观模式包含如下角色：

* Facade: 外观角色
* SubSystem:子系统角色

#### 3.2 外观模式关系图

![](/images/facaed-relative.png)

#### 3.3 代码分析

使用组合让外观（HomeTheaterFacade）能够访问子系统中所有组件，将子系统的组件整合成一个统一的接口，每项任务都是委托子系统中相应的组件处理的。

*子系统的代码没有列出。*

```
public class HomeTheaterFacade {
	Amplifier amp;
	Tuner tuner;
	DvdPlayer dvd;
	CdPlayer cd;
	Projector projector;
	TheaterLights lights;
	Screen screen;
	PopcornPopper popper;
 
	public HomeTheaterFacade(Amplifier amp, 
				 Tuner tuner, 
				 DvdPlayer dvd, 
				 CdPlayer cd, 
				 Projector projector, 
				 Screen screen,
				 TheaterLights lights,
				 PopcornPopper popper) {
 
		this.amp = amp;
		this.tuner = tuner;
		this.dvd = dvd;
		this.cd = cd;
		this.projector = projector;
		this.screen = screen;
		this.lights = lights;
		this.popper = popper;
	}
 
	public void watchMovie(String movie) {
		System.out.println("Get ready to watch a movie...");
		popper.on();
		popper.pop();
		lights.dim(10);
		screen.down();
		projector.on();
		projector.wideScreenMode();
		amp.on();
		amp.setDvd(dvd);
		amp.setSurroundSound();
		amp.setVolume(5);
		dvd.on();
		dvd.play(movie);
	}
 
 
	public void endMovie() {
		System.out.println("Shutting movie theater down...");
		popper.off();
		lights.on();
		screen.up();
		projector.off();
		amp.off();
		dvd.stop();
		dvd.eject();
		dvd.off();
	}

	public void listenToCd(String cdTitle) {
		System.out.println("Get ready for an audiopile experence...");
		lights.on();
		amp.on();
		amp.setVolume(5);
		amp.setCd(cd);
		amp.setStereoSound();
		cd.on();
		cd.play(cdTitle);
	}

	public void endCd() {
		System.out.println("Shutting down CD...");
		amp.off();
		amp.setCd(cd);
		cd.eject();
		cd.off();
	}

	public void listenToRadio(double frequency) {
		System.out.println("Tuning in the airwaves...");
		tuner.on();
		tuner.setFrequency(frequency);
		amp.on();
		amp.setVolume(5);
		amp.setTuner(tuner);
	}

	public void endRadio() {
		System.out.println("Shutting down the tuner...");
		tuner.off();
		amp.off();
	}
}
```


#### 4. 总结

* 当需要简化并统一一个很大的接口或者一群复杂的接口时，使用外观模式。
* 外观模式将客户从一个复杂的子系统中解耦。
* 实现一个外观，需要将子系统组合进外观中，然后将工作委托给子系统执行。
* 可以为一个子系统实现一个以上的外观。