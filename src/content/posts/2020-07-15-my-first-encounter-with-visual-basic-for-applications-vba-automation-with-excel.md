---
template: blog-post
title: My first encounter with Visual Basic for Applications (VBA) , automation
  with Excel
slug: automation-with-excel
date: 2020-07-18 14:23
description: My first encounter with Visual Basic for Applications (VBA) ,
  automation with Excel.
---
 This article is about the automating some manual,labouros process we tired to do always. If you think this is a new thing and you don't want to read further, believe me, you miss a powerful tool which you can develop from your very own Excel files without any dependency.

**So let's talk about the use case first**.
			As my daily development I have been coding on java with spring framework. As much we do our work is mostly on java, there is some manual part we need to spend in preparing properties files or some documents which requires for configuration of your app.  Recently I got a work to  create configuration XML files which required my main app execution.  Firstly, I don't have thought to automate it, because I'm doing it for 2 XMLs files manually for each week. Each week I'm doing 2 to 6 XML files and that piled up around 40 XMLs at the end of the 6th week. So suddenly requirements got changed and XML attributes are modified.  So doing manually all these modifications for 40 XMLs isn't the solution as I'm lazy though.

**First thoughts to automate**
Then I was trying to Automate this by creating a Java project using JAXB. But the problem I face here is, I will not maintain these XMLs after delivery. There are some Product owners has to make these changes. If I develop this tool to automate these XMLs in JAVA, I making them to install jar installations. So I need a tool to automate this without leaving from their traditional excel environment.

I was looking for some alternative and came across this hidden feature in Excel. I am not sure why they hiding this "Developer" menu item from excel ribbon by default. So after that it took some time to understand how this can be helpful for my need. Spent some time coding language of VBA. But trust me its not different from our any other language you know. 

**Lets dive into the implementation**
To get the intial peek into the output what I need , [here](https://github.com/nyalla/attachments/tree/master/generation) are the XMLs I have to generate.

If you see the XMLs most of the attribute values are static and some of them are basically generate dynamically from user choice.  

 - First step is to find out what are the static fields and what has to be dynamic.
 - Once we find out the dynamic attributes we have to make them available from some interface in excel.
In my case the attributes like country, state codes and courses are pulled out and provided a place to specify them.

**Once the above steps are clear we are ready to go for the coding part.** 

 - Open your excel, Navigate to Developer tab in the top ribbon. (If you don't find it, enable it from options)
 - Once you click on Developer tab you can find the Visual Basic Option in the first icon. Click on it and another window will open where we can write the code.
 - Just like our functions/methods, here the keyword is **sub**. 
 - First I need to tell the script to which excel sheet it has to consider. for that, the command is *Sheets("Sheet1").Select*
 Don't worry about the commands and all, whatever the thing you want to achieve search for MS documentation you will have all commands available.
 - Next, I need to tell the system to store the generated files in some specific location. 
```vb
       'Variable declaration
        Dim sDir As String
        'Get current directory
        sDir = CurDir
        
        globalDir = sDir + "\base\"
        MkDir globalDir
                
       'Display output on the screen
        MsgBox "Files will be saved in--  " & globalDir  
``` 
It will create the folder with name "base" where we can find our XMLs.

 - Next, step is to repeat the process for all available rows, something like loop through the collection items in Java. For that we have to take the last filled excel row.

 ```vb
  'To get last visible row number
    Dim lastRow As Long
    lastRow = Range("A" & Rows.Count).End(xlUp).Row
    'MsgBox lastRow
    For currentRow = 2 To lastRow
         Dim id As String
         id = Trim(Cells(currentRow, "A").Value) 
        pathName = globalDir + "\" + id + "\"
        MkDir pathName
         file1 = pathName + "DefaultList.xml"
         file2 = pathName + "DefaultPathList.xml"
         ../
         ../
         ../
Next currentRow
 ```  
Above code is something like for each loop we are iterating through all the rows.

 - Next step is to create XML structure and get the variables we need from sheet 1. In my first XML I need only one variable. 
 ```vb
Set objStream = CreateObject("ADODB.Stream")
         objStream.Charset = "iso-8859-1"
         
         objStream.Open
         objStream.WriteText ("<CusListItem>" & vbLf)
         objStream.WriteText ("  <CusItem>" & vbLf)
         objStream.WriteText ("      <Customer>Default</Customer>" & vbLf)
         objStream.WriteText ("      <CustomerId>" + id + "<Customer/Id>" & vbLf)
         objStream.WriteText ("      <CustomerCdItem>ACTIVE</CustomerCdItem>" & vbLf)
         objStream.WriteText ("      <FctvFr>2020-04-12T00:00:00</FctvFr>" & vbLf)
         objStream.WriteText ("      <ItemCd>" + id + "</ItemCd>" & vbLf)
         objStream.WriteText ("      <ItemNm>" + id + "</ItemNm>" & vbLf)
         objStream.WriteText ("      <ItemCdSummary>" + id + "</ItemCdSummary>" & vbLf)
         objStream.WriteText ("      <ItemCustomerTp>HEARD</ItemCustomerTp>" & vbLf)
         objStream.WriteText ("      <PCustomerItemIdr>CRITICAL</CustomerItemIdr>" & vbLf)
         objStream.WriteText ("      <CustomerItemCalCd>Default</CustomerItemCalCd>" & vbLf)
         objStream.WriteText ("   </CusItem>" & vbLf)
         objStream.WriteText ("</CusListItem>" & vbLf)
         objStream.SaveToFile file1, 2
         objStream.Close
```
So above block defines what has to be XML file look like. If you observe 5th  line, we are taking that **id** variable value from sheet 1  

 - Same process for next file also but with some nested XML child elements which we are taking.
```vb
 Dim countryCode As String
         countryCode = Trim(Cells(currentRow, "C").Value)
         
         Dim stateCode As String
         stateCode = Trim(Cells(currentRow, "D").Value)
         
         Dim sectionCode As String
         sectionCode = Trim(Cells(currentRow, "F").Value)
         
         Dim courses As String
         courses = Trim(Cells(currentRow, "E").Value)
         
         Dim coursesArray
         coursesArray = Split(courses, ",")
         
         For i = LBound(coursesArray) To UBound(coursesArray)
         'MsgBox coursesArray(i)
         
         objStream.WriteText ("  <ItemEndCustomerCusListItemr>" & vbLf)
         objStream.WriteText ("      <CustomeCusListItemr>Default</CustomeCusListItemr>" & vbLf)
         objStream.WriteText ("      <CustomerId>" + countryCode + "_" + stateCode + "_" + coursesArray(i) + "_" + sectionCode + "</CustomerId>" & vbLf)
         objStream.WriteText ("     <CustomerNttyStsCd>ACTIVE</CustomerNttyStsCd>" & vbLf)
         objStream.WriteText ("     <CustomerFctvItemNdPtCd>" + countryCode + "_" + stateCode + "_" + coursesArray(i) + "_" + sectionCode + "</CustomerFctvItemNdPtCd>" & vbLf)
         objStream.WriteText ("     <CustomerNdPtNm>" + coursesArray(i) + " " + sectionCode + "</CustomerNdPtNm>" & vbLf)
         objStream.WriteText ("     <CustomerIsFldDataPrst>true</CustomerIsFldDataPrst>" & vbLf)
         objStream.WriteText ("     </ItemEndCustomerCusListItemr>" & vbLf)
         
         Next
         
         objStream.WriteText ("     </ItemEndCustomerCusListItemr>" & vbLf)
         objStream.SaveToFile file2, 2
         objStream.Close
```

 - So sheet1 of my excel is like below.
 
![Excel preview](/assets/automation-with-excel-1.png "Sequence Diagram for usecase 1")


So execute  [code](https://raw.githubusercontent.com/nyalla/attachments/master/generation/xml_scripts.vba) once without making any changes, then you will understand the entire flow.

You can also link this script execution with external event also, like linking this to button click event for more user friendly. 
On Developer tab, Click on **Insert**  and select a button from **Form controls** .
After that you will get a pop listing all scripts/macros you created and select which one you want to link.

By doing this simple automation I reduced my work from 100 to 20. Apart from this we have recording option also, which will generate the written script automatically based on our actions performed on excel. This is very useful to generate basic code on what we want and to explore. 

Let me know your thoughts in the comments.
