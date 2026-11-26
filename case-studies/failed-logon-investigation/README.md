# Active Directory Failed Logon Investigation

## Scenario

I investigated whether failed logons recorded on DC01 could be linked to `testuser` in my `cyberlab.local` home lab. I used Windows Event Viewer and read-only PowerShell checks before deciding whether any changes were needed.

## What I checked

1. **Account status:** `Get-ADUser` showed `Enabled = True` and `LockedOut = False` at the start and end of the investigation.
2. **Security log:** A later 24-hour query on DC01 found three events with ID `4625` and no matches for `4740` or `4771`. This was a count for DC01's Security log, not for `testuser` specifically. One older `4625` event showed status `0xC0000133` and substatus `0x0`. Its target account name was blank, so I couldn't link it to `testuser`.
3. **Clock comparison:** From CLIENT01, I ran the command below. All three readings showed less than 1 millisecond of difference between CLIENT01 and DC01 at the time of testing. That does not establish what their clocks showed during the earlier failed logon.

   ```powershell
   w32tm /stripchart /computer:dc01.cyberlab.local /samples:3 /dataonly
   ```

## Outcome

The evidence did not link the failed logon to `testuser` or establish an account compromise. I did not reset the password, unlock the account, or change either clock.

## Evidence

![DC01 Event 4625 with status and substatus](screenshots/01-failed-logon-clock-status.png)

![Three clock offset readings from CLIENT01 to DC01](screenshots/02-client-dc-clock-offset.png)

![DC01 event count and testuser account status](screenshots/03-account-status-and-event-count.png)