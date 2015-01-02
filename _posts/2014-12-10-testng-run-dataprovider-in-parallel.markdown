---
layout: posts
title: "TestNG run dataprovider in parallel"
description: "TestNG datadriven running in parallel"
category: testng
tags: [testng, ant]
---

### References
[TestNG - Parameters with DataProviders](http://testng.org/doc/documentation-main.html#parameters-dataproviders)
[TestNG Ant Document](http://testng.org/doc/ant.html)

###parallel = true
Data providers can run in parallel with the attribute parallel. Set parallel = true for DataProvider Annotation.

```java
package demo;

import org.testng.annotations.DataProvider;
import org.testng.annotations.Test;

public class DataProviderParallelTest {

	@DataProvider(name = "initTestData", parallel = true)
	public Object[][] initTestData() {
		Object[][] testData = new Object[20][2];
		for (int i = 0; i < testData.length; i++) {
			testData[i][0] = i;
			testData[i][1] = "value" + i;
		}
		return testData;
	}

	@Test(dataProvider = "initTestData")
	public void testBody(Integer index, String value) throws InterruptedException {
		System.out.println(this.getClass().getName() + ": Thread ID: "
				+ Thread.currentThread().getId() + " Thread Name: "
				+ Thread.currentThread().getName() + " index=" + index
				+ "; value=" + value);
		Thread.sleep(2000);
	}
}
```

### Ant parameter -dataProviderThreadCount.
The number of threads to use for data providers for this run. Ignored unless the parallel mode is also specified.

#### build.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>

<project name="TestNG Demo" default="compile" basedir=".">

	<description>
		TestNG Demo
    </description>

	<property name="classes.dir" value="classes" />
	<property name="src.dir" value="src" />
	<property name="lib.dir" value="lib" />
	<property name="lib.dir" value="lib" />
	<property name="testng.report.dir" value="report" />
	<property name="test.src.dir" value="src" />
	<property name="test.dir" value="test" />

	<path id="run.cp">
		<fileset dir="${lib.dir}">
			<include name="**/*.jar" />
		</fileset>
		<pathelement location="${basedir}/classes" />
	</path>

	<taskdef name="testng" classpathref="run.cp" classname="org.testng.TestNGAntTask" />
	<taskdef resource="net/sf/antcontrib/antcontrib.properties" />

	<!--<taskdef resource="testngtasks" classpath="run.cp" />-->
	<target name="init">
		<delete dir="${classes.dir}" />
		<mkdir dir="${classes.dir}" />
	</target>

	<target name="compile" depends="init">
		<javac srcdir="${src.dir}" destdir="${classes.dir}" debug="on" includeAntRuntime="false" includejavaruntime="false" failonerror="true" nowarn="true" source="1.6">
			<classpath>
				<fileset dir="${lib.dir}">
					<include name="**/*.jar" />
				</fileset>
			</classpath>
		</javac>
	</target>

	<target name="test" depends="compile">

		<if>
			<isset property="dataProviderThreadCount" />
			<then>
				<property name="specifiedDataProviderThreadCount" value="${dataProviderThreadCount}" />
			</then>
			<else>
				<!-- if dataProviderThreadCount is not set, than set specifiedDataProviderThreadCount = 1-->
				<property name="specifiedDataProviderThreadCount" value="1" />
			</else>
		</if>

		<testng classpathref="run.cp" parallel="true" dataProviderThreadCount="${specifiedDataProviderThreadCount}" threadCount="1">
			<classfileset dir="${classes.dir}" includes="**/*.class" />
		</testng>
	</target>

</project>
```
* *ant test*
dataproviderthreadcount is not specified, set 1 as the default value, run datadriven test method one by one.
* *ant test -DdataProviderThreadCount 4*
dataproviderthreadcount is 4, using 4 threads run datadirven test method.
