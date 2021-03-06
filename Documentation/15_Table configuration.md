﻿VP Table configuration {#tableconfig}
===================

\section tableconfig_intro Introduction

To make the DirectOutput framework work, it is necessary that the framework gets information on the status changes of the elements (e.g. bumpers or lamps) on the pinball table. Depending on the type of table there a different ways how to achieve this.

If you modify a table to use the B2S.Server, the table will require the B2S.Server to be installed otherwise it will not run. Apart from the B2S.Server there are no requirements. Even if you add special commands for the DirectOutput framework to the table script, the table will run without problemes if the framework is not installed.

\section tableconfig_VPSS VP solid state (SS) tables

\subsection tableconfig_VPSSconfig Configure SS tables

Visual Pinball solid state table are using PinMame for rom and game logic emulation. Therefore all data on changed table elements is anyway sent forth and back between PinMame and Visual Pinball. This makes the necessary changes to make DirectOutput and also the B2S.Server work with the table very easy.

The only change required is to replace the statement instanciating the PinMame object:

Replace the following line

~~~~~~~~~~~~~~~{.vbs}
Set Controller = CreateObject("VPinMAME.Controller")     
~~~~~~~~~~~~~~~

with this modifed version

~~~~~~~~~~~~~~~{.vbs}
Set Controller = CreateObject("B2S.Server") 
~~~~~~~~~~~~~~~

\subsection tableconfig_VPSSextend Extend SS tables

The config explained in the previous paragraph will usually be all you have to do to make a SS table work, but you have some more options.

If you want to tell the B2S.Server and the framework about events on the table which are not reported to PinMame, you can easily add some of the EM table commands of the B2S.Server to the table to send additional information to the B2S.Server.

This mix will work without any problem as long as the table is run in a environement which has the B2S.Server installed.

Please read the section \ref tableconfig_VPEMscore for information on the EM table commands. The helpfile of the B2S Backglass Designer does also contain information on EM table commands.


\section tableconfig_VPEM VP electro mechanical (EM) tables and original tables

To make VP EM and original tables work with B2S.Server and DirectOutput, some more work is required. 

Normally these table types dont send any information to the outside world and B2S.Server as well as DirectOutput would never know about changed table element statuses. Therefore it is necessary to add some statements which will inform the B2S.Sever about the changes of table elements to the table script.

\note I recommend, that you also read the section on EM tables in the help file of the B2S Backglass Designer.

\subsection tableconfig_VPEMinit Initialization

At the beginning of the table script, usually after the Dim statements, the B2S.Server has to be instanciated and initialized. This is done with the following statements:

~~~~~~~~~~~~~~~{.vbs}
Dim Controller
Set Controller = CreateObject("B2S.Server")
Controller.B2SName = "MyTable"
Controller.Run
~~~~~~~~~~~~~~~

The B2SName which is set in the third line of code is some kind of fake romname, which will be used by DirectOutput to identify the table config in a LedControl file. Be sure to use a simple and unique name for the B2SName.

\subsection tableconfig_VPEMexit Termination

It is important that the B2S.Server and the DirectOutput framework are informed, when the user exits the table. The following code will do this:

~~~~~~~~~~~~~~~{.vbs}
Sub Table1_Exit
    Controller.Stop
End Sub
~~~~~~~~~~~~~~~

Before copying the code fragement to the table script, please check if the Table Exit section does already exist in the table. If it does, insert only the _Controller.Stop_ statement.

\warning If you dont add the _Controller.Stop_ statement to the table exit code, you might experience performance problems or instability of your cabinet! Dont forget to add it!
 
\subsection tableconfig_VPEMscore Score commands

The B2S.Server has a bunch of special commands to forward information on the scores to the B2S.Server and the backglass. The DirectOutput framework does also receive the values from these commands.

For a list and explanation of commands to set the scores, please read the section on EM tables in the helpfile of the B2S Backglass Deginer.

\subsection tableconfig_VPEMtableelements Table element status updates

\subsubsection tableconfig_VPEMtableelements_basics The basics

Apart from initializing the most important thing for the DirectOutput framework is to receive updates on table element changes.

To send this information to the B2S.Server and the framework, you have to use the _B2SSetData_ command of the B2S.Server. This commands accepts 2 parameters:

* _ID_ is the number of the table element changing its state or value. Just invent a numbering scheme of your choice for your table elements.
* _Value_ is value or status of the table element.

A complete _B2SSetData_ statement might looks as follows:
~~~~~~~~~~~~~~~{.vbs}
Controller.B2SSetData 123,1
~~~~~~~~~~~~~~~

Typically, the _B2SSetData_ satement will be put into the event handlers of the table elements. For a switch you'll maybe end up with something like this:

~~~~~~~~~~~~~~~{.vbs}
Sub sw5_Hit()
    ... some other event handling code ....
    
	Controller.B2SSetData 205,1
End Sub

Sub sw5_UnHit()
    ... some other event handling code ....
  Controller.B2SSetData 205,0
End Sub
~~~~~~~~~~~~~~~

It is important, that you also send data to the B2S.Server when the table element reverts back to it original state (e.g. in the UnHit event of a switch). If you only use a single _B2SSetData_ statement which always sends the same value for a element, the framework will not show any reaction, since the value does not change (DirectOutput reacts on value changes and NOT on the fact that data is sent).

\subsubsection tableconfig_VPEMtableelements_easy The easy way

Arngrim has developed a more comfortable way to add the code for the status updates to EM tables.

Instead of using the Controller.B2SSetData command (explained in the previous section) you might add one of the following code segments to your table script:

<b>Version for tables which do always use the B2S.Server</b>
~~~~~~~~~~~~~~~{.vbs}
'*DOF method for non rom controller tables by Arngrim****************
'*******Use DOF 1**, 1 to activate a ledwiz output*******************
'*******Use DOF 1**, 0 to deactivate a ledwiz output*****************
'*******Use DOF 1**, 2 to pulse a ledwiz output**********************
Sub DOF(dofevent, dofstate) 
	If dofstate = 2 Then
		Controller.B2SSetData dofevent, 1:Controller.B2SSetData dofevent, 0
	Else
		Controller.B2SSetData dofevent, dofstate
	End If
End Sub
'********************************************************************
~~~~~~~~~~~~~~~

<b>Version for tables which where B2S support can be turned on or off</b>
Requires the variable <i>B2SOn</i> to be set to true or false to enable/disable B2S support.
~~~~~~~~~~~~~~~{.vbs}
'*DOF method for non rom controller tables by Arngrim****************
'*******Use DOF 1**, 1 to activate a ledwiz output*******************
'*******Use DOF 1**, 0 to deactivate a ledwiz output*****************
'*******Use DOF 1**, 2 to pulse a ledwiz output**********************
Sub DOF(dofevent, dofstate) 
	If B2SOn=True Then
		If dofstate = 2 Then
			Controller.B2SSetData dofevent, 1:Controller.B2SSetData dofevent, 0
		Else
			Controller.B2SSetData dofevent, dofstate
		End If
	End If
End Sub
'********************************************************************
~~~~~~~~~~~~~~~


If these code segements are added, you can simplay call the command DOF to submit status updates to the Direct Output framework.

You got 3 options:

* __DOF {TableElementNumber},0__ sends a off state for the table element to DOF.
* __DOF {TableElementNumber},1__ sends a on state for the table element to DOF.
* __DOF {TableElementNumber},2__ sends a on state, immediately followed by a off state to DOF.

For more details on the methode please read Arngrims or post over at http://vpuniverse.com/forums/topic/1105-easy-methods-to-dof-an-original-table/

\section tableconfig_nobackglass Tables w/o B2S.Server Backglass

There are still some tables around which dont have a backglass running with B2S.Server. If the B2S.Server is instanciated in such a table, it will normaly complain about the missing backglass file and therefore stop you from running the B2S.Server and the DirectOuput framework plugin.

To overcome the problem, uncheck the "Error message without backglass" checkbox in the B2S.Server settings window.

\image html B2SServer_Settings_NoBackglass.png Error message without backglass checkbox.

\note For tables without a backglass you will not be able to call the settings window of the B2S.Server.

Alternatively it is also possible to edit the B2STableSettings.xml file in the table directory. You will need to add the following line to your B2STableSettings.xml file:

~~~~~~~~~~~~~~~{.xml}
<B2STableSettings>
  ....
  <ShowStartupError>0</ShowStartupError>
  ...
</B2STableSettings>
~~~~~~~~~~~~~~~

This will stop the B2S.Server from showing the error message if no backglass file is found.

Dont forget that you will have to replace the line

~~~~~~~~~~~~~~~{.vbs}
Set Controller = CreateObject("VPinMAME.Controller")     
~~~~~~~~~~~~~~~

with this modifed version

~~~~~~~~~~~~~~~{.vbs}
Set Controller = CreateObject("B2S.Server") 
~~~~~~~~~~~~~~~


Using this option any table can use the B2S.Server no matter wether it has a backglass or not. Even tables with a classic Rosve style, no B2S.Server, legacy backglass can use the B2S.Server and its plugins using the mentioned option.

Apart from setting the mentioned option in the B2STableSettings.xml file, configuring the table for DirectOutput and B2S.Server is exactly the same as explained in the sections above.