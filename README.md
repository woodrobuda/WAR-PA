Weekly Activity Report (WAR) PowerApp Tool - Data entry application for team work logging with Excel backend
Sheet 1: "ActivityLog"
Columns (in order):
- EndOfWeekDate (Date format: MM/DD/YYYY)
- Analyst (Text)
- ISSO Task (Text)
- Key Accomplishments/Activities (Text)
- Key Meetings (Text)
- Next 7 Days of Work Planned (Text)

Sheet 2: "DropdownLists"
Columns:
- task (Column A)
- analyst (Column B)

Data rows:
task                     | analyst
ERA                      | Roy Woods
ORR                      | Oge Ogele
ATO activity support     | Don Hale
ISSO Support             | Thunder Sargent
Zone Architecture        | Allen Molthan
                         | Megan Stout


           Action Steps for Excel Setup:

Add a header row to ActivityLog with the 6 column names above
Format ActivityLog as an Excel Table: Select data → Insert → Table → Name it ActivityLog
Format DropdownLists as an Excel Table: Select data → Insert → Table → Name it DropdownLists
Save and upload to OneDrive (shared folder)

PART 2: POWERAPP BUILD (FULLY CODED)
Step 1: Create the PowerApp & Connect to Excel
Go to PowerApps Studio
Click Create > Canvas app from blank
Name it: WeeklyActivityReport
Tablet or Phone layout (recommend Tablet for more screen space)
Once loaded, go to Data > Add data > Excel Online (Business)
Select your OneDrive folder → WAR_Database.xlsx
Select both tables: ActivityLog and DropdownLists
Click Connect

Step 2: Add Form Controls to the Canvas
Screen layout:
Create a vertical layout with:

Title label (at top)
Date Picker
Analyst Dropdown
Task Type Dropdown
Three text input boxes (Accomplishments, Meetings, Next Week Plan)
Character count labels (under each text box)
Submit button
Success/Error notification area

Step 3: Add Controls & Set Formulas
Control 1: Title Label


Code
Label: "Weekly Activity Report (WAR)"
Font Size: 28
Font Weight: Bold
Text Color: Dark Blue (#003366)
Control 2: Date Picker

Add a DatePicker control (from Insert > Input):

Code
Name: DatePicker_EndOfWeek
Label: "End of Week Date:"
Format: DateFormat.ShortDate (displays as MM/DD/YYYY)
Default: Today()
Control 3: Analyst Dropdown

Add a Dropdown control:

Code
Name: Dropdown_Analyst
Label: "Analyst Name:"

Items property (formula):
Sort(
  Filter(
    DropdownLists,
    !IsBlank(analyst)
  ).analyst,
  Ascending
)

OnChange: Set(selectedAnalyst, Self.Selected.Value)
Control 4: Task Type Dropdown

Add another Dropdown control:

Code
Name: Dropdown_Task
Label: "ISSO Task:"

Items property (formula):
Sort(
  Filter(
    DropdownLists,
    !IsBlank(task)
  ).task,
  Ascending
)

OnChange: Set(selectedTask, Self.Selected.Value)
Control 5: Accomplishments Text Input

Add a Text input control (multiline):

Code
Name: TextInput_Accomplishments
Label: "Key Accomplishments/Activities:"
Mode: MultiLine
Height: 120
MaxLength: 500
PlaceholderText: "Enter up to 500 characters..."
BorderColor: If(Len(Self.Text) > 500, Red, LightGray)
Add a label below it for character count:

Code
Name: Label_AccomplishmentsCount
Text: Len(TextInput_Accomplishments.Text) & "/500"
Font Size: 11
Font Color: If(Len(TextInput_Accomplishments.Text) > 500, Red, Green)
Control 6: Meetings Text Input

Add another Text input control (multiline):

Code
Name: TextInput_Meetings
Label: "Key Meetings:"
Mode: MultiLine
Height: 120
MaxLength: 500
PlaceholderText: "Enter up to 500 characters..."
BorderColor: If(Len(Self.Text) > 500, Red, LightGray)
Add character count label:

Code
Name: Label_MeetingsCount
Text: Len(TextInput_Meetings.Text) & "/500"
Font Size: 11
Font Color: If(Len(TextInput_Meetings.Text) > 500, Red, Green)
Control 7: Next 7 Days Text Input

Add another Text input control (multiline):

Code
Name: TextInput_Next7Days
Label: "Next 7 Days of Work Planned:"
Mode: MultiLine
Height: 120
MaxLength: 500
PlaceholderText: "Enter up to 500 characters..."
BorderColor: If(Len(Self.Text) > 500, Red, LightGray)
Add character count label:

Code
Name: Label_Next7DaysCount
Text: Len(TextInput_Next7Days.Text) & "/500"
Font Size: 11
Font Color: If(Len(TextInput_Next7Days.Text) > 500, Red, Green)
Control 8: Submit Button

Add a Button control:

Code
Name: Button_Submit
Text: "Submit"
Fill: Green (#107C10)
TextColor: White
OnSelect: (see formula below)
OnSelect Formula (CRITICAL - File Lock Handling):

powerapps
If(
  Len(TextInput_Accomplishments.Text) > 500 Or
  Len(TextInput_Meetings.Text) > 500 Or
  Len(TextInput_Next7Days.Text) > 500,
  (
    Notify("Error: One or more fields exceeds 500 characters.", NotificationType.Error);
    Set(formValid, false)
  ),
  If(
    IsBlank(Dropdown_Analyst.Selected) Or
    IsBlank(Dropdown_Task.Selected) Or
    IsBlank(DatePicker_EndOfWeek.SelectedDate),
    (
      Notify("Error: Please complete all required fields.", NotificationType.Error);
      Set(formValid, false)
    ),
    (
      Set(formValid, true);
      Set(isSubmitting, true);
      
      If(
        IsError(
          Patch(
            ActivityLog,
            Defaults(ActivityLog),
            {
              EndOfWeekDate: DatePicker_EndOfWeek.SelectedDate,
              Analyst: Dropdown_Analyst.Selected.Value,
              ISSO_Task: Dropdown_Task.Selected.Value,
              'Key Accomplishments/Activities': TextInput_Accomplishments.Text,
              'Key Meetings': TextInput_Meetings.Text,
              'Next 7 Days of Work Planned': TextInput_Next7Days.Text
            }
          )
        ),
        (
          Notify("⚠️ Submission failed: Data file is currently locked by another user. Please wait a moment and try again.", NotificationType.Warning);
          Set(isSubmitting, false)
        ),
        (
          Notify("✓ Record submitted successfully!", NotificationType.Success);
          
          Reset(TextInput_Accomplishments);
          Reset(TextInput_Meetings);
          Reset(TextInput_Next7Days);
          Reset(Dropdown_Analyst);
          Reset(Dropdown_Task);
          Reset(DatePicker_EndOfWeek);
          
          Set(isSubmitting, false)
        )
      )
    )
  )
)
Control 9: Retry Button (Optional)

Add a second button for users who get a lock error:

Code
Name: Button_Retry
Text: "Try Again"
Fill: Orange (#F7630C)
Visible: !IsBlank(Label_ErrorMessage.Text)
OnSelect: Button_Submit.OnSelect
Step 4: Add Error Message Display
Add a Label control at the bottom:

Code
Name: Label_ErrorMessage
Text: ""
Font Size: 14
Font Color: Red
Visible: !IsBlank(Self.Text)
Step 5: Configure App OnStart (Optional but Recommended)
Go to App > OnStart property:

powerapps
Set(isSubmitting, false);
Set(formValid, false);
Set(selectedAnalyst, Blank());
Set(selectedTask, Blank())
This initializes variables when the app loads.

PART 3: DEPLOYMENT TO ONEDRIVE (STEP-BY-STEP)
Step A: Prepare OneDrive Folder
Create a shared OneDrive folder:

Go to OneDrive
Create new folder: WAR_Tool_Data
Right-click → Share
Share with your team (or AD group)
Set permissions to "Edit"
Upload Excel file:

Upload WAR_Database.xlsx to this folder
Confirm both sheets (ActivityLog and DropdownLists) are properly formatted as Tables
Step B: Publish PowerApp
In PowerApps Studio, click File > Save
Click Publish
Click Publish this version
Wait for completion
Step C: Share PowerApp with Team
In PowerApps Studio, click Share (top-right)
Enter team member emails or AD group name
Set permission: Use (not Edit, unless you want them modifying the app itself)
Click Share
Step D: Test the App
Have 2-3 users open the WAR app
Submit test records from each user
Check that records appear in ActivityLog worksheet in Excel
Test the "lock" message by having 2 users submit simultaneously
PART 4: ADMIN GUIDE (MAINTAINING LOOKUP LISTS)
Adding a New Analyst:
Open WAR_Database.xlsx on OneDrive
Go to DropdownLists worksheet
Find the next empty row in column B (analyst)
Type the new name (e.g., "Jane Smith")
Save the file
Changes appear in PowerApp dropdowns within seconds
Adding a New Task Type:
Open WAR_Database.xlsx
Go to DropdownLists worksheet
Find the next empty row in column A (task)
Type the new task (e.g., "Risk Assessment")
Save the file
Changes appear in PowerApp dropdowns within seconds
Reviewing Submitted Records:
Open WAR_Database.xlsx
Go to ActivityLog worksheet
Filter/sort by:
End of Week Date
Analyst Name
Task Type
Export as PDF or create a pivot table for management reports
PART 5: SIMPLE USER GUIDE (FOR YOUR TEAM)
Quick Start: How to Submit Your Weekly Activity Report
1. Open the App

Find and click "Weekly Activity Report (WAR)" in PowerApps
Or access via: [Your PowerApps Link]
2. Fill in the Form:

Field	What to Do
End of Week Date	Click the calendar icon, select Friday of the week you're reporting
Analyst Name	Drop down and select your name
ISSO Task	Drop down and select the primary task you worked on (e.g., "ORR", "ISSO Support")
Accomplishments	Type key activities you completed (max 500 chars). Use bullet points for clarity.
Meetings	List meetings attended with dates (max 500 chars).
Next 7 Days	List planned work for the coming week (max 500 chars).
3. Submit:

Click the green "Submit" button
You'll see a success message: ✓ Record submitted successfully!
The form clears automatically for your next entry
4. If You Get an Error:

"File is locked" message? → Wait 30 seconds and click "Try Again"
"Complete all fields" message? → Make sure all dropdowns are selected and text areas have content
TROUBLESHOOTING
Issue	Solution
Dropdown is empty	Check DropdownLists table in Excel has data; make sure it's formatted as a Table
Submit fails every time	Confirm Excel file is on OneDrive and you have "Edit" access
Data doesn't appear in Excel	Click "Refresh" in the app or close/reopen Excel file
Character count shows wrong number	Ensure text input is in MultiLine mode
File Summary
Here's a complete file reference:

name=WAR_Database_Template.xlsx
SHEET 1: ActivityLog
[Table: ActivityLog]

EndOfWeekDate | Analyst | ISSO Task | Key Accomplishments/Activities | Key Meetings | Next 7 Days of Work Planned
02/27/2026    | Don Hale| ORR      | (text data)                   | (text data)  | (text data)

---

SHEET 2: DropdownLists
[Table: DropdownLists]

task                  | analyst
ERA                   | Roy Woods
ORR                   | Oge Ogele
ATO activity support  | Don Hale
ISSO Support          | Thunder Sargent
Zone Architecture     | Allen Molthan
                      | Megan Stout
