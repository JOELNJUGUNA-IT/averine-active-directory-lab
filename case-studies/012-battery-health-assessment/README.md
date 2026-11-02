# Case Study 012: Windows Battery Health Assessment

## Objective

Assess a Windows laptop battery using built-in tools while protecting sensitive information.

## Environment

- Windows 11
- Acer Aspire A15-51M
- Windows PowerShell 5.1
- Lithium-polymer battery

## Procedure

1. Opened Windows PowerShell.
2. Generated the report using `powercfg /batteryreport`.
3. Opened it using `start .\battery-report.html`.
4. Recorded the battery capacity and cycle count.
5. Removed identifying information from the evidence.

## Results

- Design capacity: 52,996 mWh
- Full-charge capacity: 50,176 mWh
- Cycle count: 23
- Chemistry: LiP
- Estimated battery health: 94.7%
- Estimated wear: 5.3%

## Conclusion

The battery is healthy. Its full-charge capacity remains approximately 94.7% of its original capacity, so replacement is not currently required.

## Skills Demonstrated

- PowerShell
- Windows hardware diagnostics
- Battery-health calculations
- Technical documentation
- Privacy-conscious evidence handling

## Evidence

![Battery-health results](screenshots/battery-health-results.png)