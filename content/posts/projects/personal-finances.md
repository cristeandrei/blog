---
title: "Personal finances PWA"
date: 2022-07-22
tags: ["redux", "javascript", "react", "firebase","project"]
author: "Andrei Criste"
draft: false
summary: "Mobile first PWA for keeping track of expenses and income, and for providing other useful functionalities."
---

## About the app

Mobile first PWA for keeping track of expenses and income. Among its functionalities are: entries can be organised in
accounts, files can be attached to entries, each account can be organised in multiple sub-accounts and so forth for
better organization, common accounts so multiple users can add entries, notifications in case the accounts reached a
certain sum, set notifications that trigger at a chosen interval for invoice remainders, creates useful graphs for
tracking spending.

[Github Repo](https://github.com/CristeAndrei/budgetix)

This was my first big application, because of that and my lack of foresight I didn't documented the development process,
so all I have to offer is a github repo with a bunch of questionable code( don't worry I got better, probably), a
user guide for a app that will never be deployed again and a schema of the data that used to be store on Firestore. With
all that I think its fitting that my first post here to be the first more complex app I made.

## Technologies and libraries used

- React
- Material UI
- Redux Toolkit
- React Router DOM
- Firebase Authentication
- Firestore
- Firebase Functions
- Firebase Messaging
- Firebase Store
- Workbox
- React Chartjs 2
- MomentJs
- React Material UI Form Validator
- React Icons
- Uuid

## Database

This is how the data stored in Firestore looks looks like. We have 5 collections, the collections called Fluxes and
Lines represents the accounts and entries that we described in the About section.

![database](https://user-images.githubusercontent.com/60604692/169547171-9059b978-d903-4f4f-a199-92dd143e8fa8.png)

### Fluxes

Fluxes are used to group our entries. The balance field represents the sum of all the entries in a specific flux, the
totalBalance field takes in considerations the entries from fluxes nested in a flux. This two fields are updated each
time we modify, delete or add entries. The path array stores data about the path to we must traverse to reach the
current flux from the root flux, this is used for generating breadcrumbs for a easier navigation between fluxes. There
are two types of
fluxes, fluxes where just one user can add manage them and fluxes that can be managed by multiple users. We can tell
witch type of
flux we are dealing with by checking the type field, for single users we have the 'flux' string and for multi user ones
we
have the 'group' string. The userId field keeps track of all the users that have access to a flux. When adding a user to
a flux, we also add it to all the nested fluxes of that flux. The subscribedBudgets filed keeps track of the budgets
that one
flux is part of.

### Lines

The only thing special about the lines collections are the fileName and url fields. They are present only when we have
entries
that we attached a file to. The url links to our storage in Firebase Storage.

### Graphs

When we create a graph there are two things that we can modify to get our desired graph. The data that we want to use,
this is done
by providing the list of fluxes from witch we want to get data from and storing it in fluxList, and a time range so if
we want we can only get data from a
specific period stored in the fromDate and toDate fields. Additionally we can chose how our data is grouped in the
timeFormat field, we have
three options: days, months and years. For example if we chose months all the entries in a month is added up and used
when constructing our graph.

### Users

Additionally to the usual things that we store in a users table, here we also store tokens. A token is generated when we
login, updated in
the database and cleared when we logout. They are used in serverless functions to send notifications to users on all the
devices they are logged
in.

### Budgets

Notting special , just data about what limit we set for our budget, the users that have access to the budget, a detailed
list with
all the fluxes subscribed to the budget and a totalBalance of the budget.

### Tasks

Now here is a more interesting collection. This tasks are queried every minute from a Firebase Function program so we
can send notifications
to users that got past their limit on one budget( in case the budget has multiple users they are notified too) or to
users that have set timed
remainders( here we can also have multiple notifications). Because of that in the type field we can have "
budgetNotification" or "invoiceNotification". The worker field stores what kind
of notification we send, we have notifications that are send just to a single user called "sendDeviceMessage" and
notifications to multiple
users called "sendMultipleDeviceMessage". The status field can be "scheduled" if every thing is ok and the notification
is scheduled to be
send eventually or "error" if something has happened and i couldn't be send. This is useful because the user will be
made aware in the UI and
can reset the notification. PerformAt is just the that when the notification will be send. The options object stores
data about the interval at
witch the notification will be send, its content and to whom we will send the notification to.

## Authentication

In-app authentication can be done in two ways depending on your user account, either using your account email and
password, or using your Google account by pressing the Google logo button.

![auth](https://user-images.githubusercontent.com/60604692/159226301-262780ff-18d2-4796-b0af-58ae9fa73b38.jpg)

## Creating an account

There are two types of user accounts, one is created through an email address and a password, and the other is through a
Google Account. The difference between the two types of accounts is that when choosing to create an account using an
email address, the user chooses their username directly, unlike accounts created using the Google platform, the username
of these users being the gmail address. If you want to change your username, you can do this in the account screen. The
email address and username are unique to each user. Creating an account does not require email verification, it is only
used for password recovery.

![signin](https://user-images.githubusercontent.com/60604692/159226302-bf4df1bc-3752-479c-9d70-9fc1556653c5.jpg)

## Password reset

Resetting the password is done by clicking "Forgot Password? ” in the login screen. In case of a successful reset an
email will be sent to the specified email address with instructions for resetting the password.

![password-reset](https://user-images.githubusercontent.com/60604692/159226300-8bd33691-3207-42b0-8e3d-9cc9cfeeb8d9.jpg)

## Home screen

After logging in we are redirected to the home screen of the application. From this screen we can navigate the fluxes we
create, add entries to them, edit existing fluxes and entries, and add budget or chart subscriptions to fluxes.

### Bottom actions

In the bottom bar of the application are the buttons used for performing actions on the current flux or for displaying
information about it. Holding down one of these buttons displays a tooltip with a description of the action performed by
each button.

![start-screen](https://user-images.githubusercontent.com/60604692/159226308-dd21a1b5-119c-4c57-bcd8-4256fb6c510b.jpg)

## Creating a flux

Clicking on the "Add" icon in the lower right corner opens the menu for actions that can be performed on a flux. Then
press the "Add Flux" button.

![create-flux](https://user-images.githubusercontent.com/60604692/159226310-d69c1439-11a4-4549-b08a-a92051d93d53.jpg)

It opens the flux creation dialog. Here you can select the name of the stream and whether it should be added to the
budgets to which the parent stream is subscribed (if the parent stream does not belong to any budget this option is not
available).

![create-flux-dialog](https://user-images.githubusercontent.com/60604692/159226306-08b8e400-5b1a-4f09-8471-24c8698713d6.jpg)

## Flux navigation

If there are fluxes that can be navigated then a menu called "Fluxes" appears under the application bar, when pressed it
expands, displaying all the fluxes that can be navigated from the current flux. Clicking on such a flux redirects us to
the selected flux.

![flux-nav](https://user-images.githubusercontent.com/60604692/159226311-d7d00ea3-2abd-457e-9cf8-0718a8b56d7a.jpg)

![extended-nav](https://user-images.githubusercontent.com/60604692/159226317-c4d53a9f-483c-4675-8ffd-0a6dd0151724.jpg)

To navigate back to the fluxes we’ve been in, we use the list of links above the "Fluxes" menu. When we click on such a
link we are redirected to that flux.

![breadcrums](https://user-images.githubusercontent.com/60604692/159226316-f9f3a5b5-d1e5-465a-8fc8-ca14ec08361a.jpg)

## Flux info and edit dialog

The menu for editing and displaying information is opened by pressing the "Info Flux" button accessible from the bottom
action bar of the application in the "Info" menu.

![flux-info-button](https://user-images.githubusercontent.com/60604692/159226313-402b9003-beb0-45df-9cb5-92c3f61f1a5e.jpg)

In this menu you can see the date of creation of the flux, the name, the total value of the entries strictly in this
flux as well as the total value of the entries in this flux and in the fluxes belonging to it.
Pressing the "Edit" button makes it possible to edit the flux name. The "Delete Flux" button deletes the flux, the
entries associated with it, and any subscription it has of one of the budgets or charts created. Deleting a flux also
triggers deleting fluxes that belong to it. If we want to delete only the records of the selected flux we can use the "
Delete Lines" button.

![flux-info-dialog](https://user-images.githubusercontent.com/60604692/159226314-b1f40a5d-7760-4a08-acb2-084e3919f49c.jpg)

## Creating an entry

There are two types of entries that can be created. Simple, they are defined by a entrie value and a name. In addition,
there is the option of entries with an attached file. The dialog for creating any type of entrie can be accessed from
the "Add" button at the bottom right.

![add-simple-entrie](https://user-images.githubusercontent.com/60604692/159226327-bdf09229-02e1-4e14-8f67-21a95a2a6ef1.jpg)

![add-file-entrie](https://user-images.githubusercontent.com/60604692/159226328-6b2ac2ee-d67c-40b2-8286-4e8c9ea12ad6.jpg)

![simple-entrie-dialog](https://user-images.githubusercontent.com/60604692/159226325-5df5eaa4-a4fb-46a5-bb37-20ead6e400a2.jpg)

![file-entrie-dialog](https://user-images.githubusercontent.com/60604692/159226323-a1a73a9a-3ca8-4f52-84d0-7537180f8328.jpg)

## File progress

All files being uploaded can be seen in the lower left corner of the "Pending Files" menu. The number of files being
uploaded is displayed in the corner of the "Pending Files" icon. When pressed, a list of file progress is displayed.

![progress-bar](https://user-images.githubusercontent.com/60604692/159226329-c2360a8d-89e2-4c41-94f8-9821d3905340.jpg)

## Entry edit

After creating a entrie, it will be displayed under the flux navigation menu. Clicking on an entry opens a dialog that
displays information about it and makes it possible to edit or delete it. In case of editing, its name and value can be
changed. Entries that are accompanied by a file provide an "Open Link" link for downloading that file.

![line-info](https://user-images.githubusercontent.com/60604692/159226321-47f34987-0320-4dff-b785-a7369e1e7e45.jpg)

## Navigation

Navigation is done through the application sidebar. Clicking on the icon in the upper left corner (when using mobile
devices can also swipe from left to right), opens the menu where all the application screens are listed. By clicking "
Budgetix" in the application bar we are redirected to the home page.

![side-nav](https://user-images.githubusercontent.com/60604692/159226330-bf0e404f-eda7-49be-a39a-cdc1423e65f4.jpg)

## Multi user fluxes

From the navigation menu you can access the fluxes for groups, the difference between them and the normal fluxes is the
possibility to add other users to them. All users who belong to a flux can make changes to it and add or remove other
users. Subfluxes created in such a flux can be accessed by all users of the parent flux.

![group-button](https://user-images.githubusercontent.com/60604692/159226259-c573485e-8829-4450-ba02-5535f7c9c0fa.jpg)

## Adding users to multi user fluxes

To add a user to a multi user flux, click on the "Add" menu in the lower left corner followed by pressing the "Subscribe
User" button. This button opens a dialog where we can enter the username of an existing user to add it to the flux.

![group-dialog](https://user-images.githubusercontent.com/60604692/159226266-f21a9eff-9278-421e-863a-13aa268c4e6b.jpg)

## Removing users from multi user fluxes

To remove a user added to a multi user flux, click on the "Info" menu in the middle of the bottom bar of the application
followed by pressing the "Subscribed Users" button. This button opens a dialog containing a list of all users who have
access to the flux we are in. By selecting one or more users and pressing "Delete" we can remove users from the current
flux. If a user is also subscribed to a budget in the flux from which it was removed, it is also removed from that
budget.

## Budgets

Clicking on the "Budgets" icon in the left navigation menu accesses the budget screen. Here is a list of all the budgets
you belong to and a way to create new budgets by pressing the “+” button at the bottom right.

![budgets-screan](https://user-images.githubusercontent.com/60604692/159226269-8f473d4a-c73a-4cd5-8181-27fb751519ed.jpg)

## Creating a budget

Pressing the “+” button at the bottom right opens the budgeting dialog. Here you can choose the name of the budget, its
limit and users who want to be part of this budget.

![create-budget](https://user-images.githubusercontent.com/60604692/159226272-17d4d62d-b3e5-4759-87ca-4fd982e386bc.jpg)

## Edit or delete a budget

Pressing the button with the pencil icon next to a budget opens the budget editing dialog. Here we can edit the budget
name, its limit, and users who have access to the budget, or we can delete the budget. If we have single user fluxes
subscribed to this budget then we can't add other users.

![edit-budget](https://user-images.githubusercontent.com/60604692/159226274-3b338142-75e7-42c6-8443-7bf148606e87.jpg)

## Adding fluxes to budgets

Adding a flux to the budget is done through the "Add" menu on the main screen, by pressing the "Subscribe to budget"
button. It opens a menu where we can see all the budgets we can subscribe this flux to and the budgets that the flux is
already subscribed to.

![subscribe-budget-button](https://user-images.githubusercontent.com/60604692/159226278-529d140a-0f85-4714-8f54-2905d046ba4b.jpg)

At the end when you click "Submit" the budgets that are new in the "Chosen" column will be added to the flux
subscription list and the budgets that were in the "Chosen" column but are now in the "Choices" column will be removed
from the flux subscription list. We can also choose whether we want to subscribe only the fluxes we are in or whether we
want to add the fluxes that belong in the current flux(note that all fluxes that start from the current flux are added)
. If the budget has more users, they are added to the fluxes. For the fluxes that only we have access to, the budgets
that have multiple users are not displayed.

![budget-dialog](https://user-images.githubusercontent.com/60604692/159226275-71768cd7-606d-46ab-8f3c-c5b5ef0200f3.jpg)

## Flux info related to budgets

Through the "Info" menu at the bottom of the application we can click on the "Budgets Subscriptions" button.

![budget-subscriptions-button](https://user-images.githubusercontent.com/60604692/159226277-308b4315-6fad-4dbc-8166-b02b0e41ff08.jpg)

Here we have a list of all the budgets to which a flux is subscribed. When we tap a budget in the list, we are
redirected to the budget information screen.

![budget-subscription-list](https://user-images.githubusercontent.com/60604692/159226280-c27fbed1-4ead-4eff-b5a1-0604b40528d5.jpg)

## Budget info

We can access this screen in two ways, through the "Info" menu at the bottom of the application we can click on the "
Budgets Subscriptions" button or from the left navigation menu by pressing the "Budgets" button and selecting a budget
from the list. A graphical representation of the fluxes subscribed to the budget can be seen in this screen and also a
graphical representation of the relationship between the sum of all fluxes and the set limit represented in percentages.
Budget information such as its limit and the amount of fluxes that are subscribed to it. A list of all the fluxes and
their value. We can remove a flux from the budget by pressing the "x" button to the right of a stream. Here you can also
see the notification that is automatically received when we exceed the set limit. It can be edited here by pressing the
button represented by a pencil next to the notification.

![budget-screen](https://user-images.githubusercontent.com/60604692/159226282-22f6fea6-9d6f-4d38-92ff-e5846f857f09.jpg)

## Creating a graph

Graphics are created in the graphs screen, which can be accessed through the sidebar by clicking on the icon named "
Graphs". There we have a list of all already created graphs.

![graph-screen](https://user-images.githubusercontent.com/60604692/159226283-284a4ab6-59e9-4b44-904c-df8213ed79d8.jpg)

Creating a graph is done by pressing the button with the icon represented by an "+". This button opens a dialog where we
can enter the desired name for the created graph.

![graph-create](https://user-images.githubusercontent.com/60604692/159226285-4bb0ce21-c24d-46b3-8a11-2b15005b0d64.jpg)

Editing a created graph is done by pressing the button represented by the pencil icon. The open dialog makes it possible
to edit a graph name or delete it. Clicking on a graph in the list opens the screen with information about it, its
settings and the graph itself.

![graph-edit](https://user-images.githubusercontent.com/60604692/159226288-0615ecc3-a19f-4554-ba1f-a66ba088c9d6.jpg)

## Adding a flux to a graph

Subscribing a flux to the graph is done in the same way as subscribing flux to a budget. Through the "Add" menu,
pressing the "Subscribe to graph" button.

![subscribe-graph-button](https://user-images.githubusercontent.com/60604692/159226281-1c7bd260-cedb-44a1-a9f9-6ca3bd086cc1.jpg)

![graph-subscription](https://user-images.githubusercontent.com/60604692/159226289-cf8164d8-84ad-4f20-8248-1a411f652a97.jpg)

## Graphs and graphs settings

To adjust the setting of a graph or to view it, select it in the Graphs screen. Graphs can be set to take only data from
a specific period from subscribed fluxes. In addition, you can set how the flux data is calculated. There are three
calculation options: grouping by day, grouping by month and grouping by year. Grouping means that for a point on the
graph we use the sum of all records in a day, month, or year, depending on the setting chosen. The graphs show the total
value of the flux at a given date and the value with which it increased or decreased. There is also a list of fluxes on
this page, you can delete a flux by pressing the minus button in a black circle next to it.

![graph](https://user-images.githubusercontent.com/60604692/159226291-7020555d-bda7-42b1-ae74-e99cb1bd3be0.jpg)

## Notifications

The notifications screen can be accessed from the sidebar.

![notifications](https://user-images.githubusercontent.com/60604692/159226292-2df03401-6d8d-4bdc-bc29-d2de6c7a428d.jpg)

Here you can create new notifications, edit existing notifications, put certain notifications on hold, or delete
notifications. There are two types of notifications, user-created, or automatically created when a budget is exceeded.
To create a notification we must provide the date we want to receive the notification, the notification message, and the
number of days it must pass before the notification is resend. The creation is done by pressing the button on the bottom
right represented by a bell with a plus in the middle.

![create-notification](https://user-images.githubusercontent.com/60604692/159226296-efbd5edb-cf10-4c06-9875-a2e1426aac4d.jpg)

To edit, you can click on a notification in the list. To disable a notification we use the switch to the right of it.The
switch is replaced by an exclamation point in a black circle in case of an error, it is recommended to delete the
notification and create a new notification.

![edit-notification](https://user-images.githubusercontent.com/60604692/159226293-24dfdc09-a2ce-4dea-899f-84eae997abc9.jpg)

## Edit account info

The user account can be edited via the "Account" button in the left navigation menu. You can change your username for
both types of accounts. For accounts created using email and password, it is possible to change the password or email.
Editing is done by clicking on the desired field, this action triggers the opening of a dialog in which we can enter the
new info. The password must also be entered to edit the email.

![email-account](https://user-images.githubusercontent.com/60604692/159226299-59820175-2819-4fb0-b357-5b76770f7337.jpg)

![gmail-account](https://user-images.githubusercontent.com/60604692/159226297-184c9375-f639-4246-b639-b23aee3c5165.jpg)

## Exit the app

Exiting the application is done by pressing the "Logout" button in the sidebar.
