BASH, cURL, and Log Analysis Walkthrough Writeup
(With explanations + pentest-report-ready language)
Exercise 1: Basic BASH Queries & Data Extraction
Objective:

Gain familiarity with BASH commands used for data extraction, system enumeration, filtering, and text processing.

1. File Preparation

The environment already contains wordlists.txt and duplicate copies inside the home directory. These files serve as sample data for text extraction and parsing.

2. Extracting Fields from a File
Command
cut -d' ' -f2 file.txt

Explanation

cut extracts specific fields from text.

-d ' ' sets the delimiter as a space.

-f2 selects the second field.

Pentest relevance: Useful when parsing leaked credential dumps, CSV exports, log files, etc.

3. CPU Information Enumeration
Command
cat /proc/cpuinfo

Explanation

Reads kernel-provided CPU details.

Common during system profiling or reconnaissance.

Your machine’s microcode entry:

microcode : 0xffffffff

4. Recursive File Size Listing + Sorting
Command
sudo ls / -R -s | sort -n -r | head -5

Explanation

ls -R recursively lists directory contents.

-s displays file size blocks.

sort -n -r sorts numerically in reverse → largest first.

head -5 shows top 5 items.

Pentest relevance: Quickly identifies large or suspicious files during investigations or post-exploitation.

5. Find Recently Modified Files
Commands
sudo find /home -mmin -5
sudo find /home -mtime -1

Explanation

-mmin -5 = modified within last 5 minutes

-mtime -1 = modified within last 24 hours

Pentest relevance: Investigates recent user activity or malware behavior.

6. Hex, Binary, and ASCII Transformation
Hex Dump with Offset
sudo xxd -s 35 -l 50 wordlists.txt


xxd → hex dump tool

-s offset

-l length

ASCII to HEX
printf 'A' | xxd

HEX to ASCII
printf 0x41 | xxd -r

ASCII to Binary
printf 'A' | xxd -b


Pentest relevance:
Used in payload encoding, reverse engineering, and binary inspection.

7. Extract Printable Strings from a Binary
Command
sudo egrep -a -o '\b[[:print:]]{2,}\b' /sbin/ifconfig


Explanation:

-a treats binary as text

-o prints only matches

regex extracts printable strings

Remove duplicates
sudo egrep ... | sort -u


Pentest relevance:
Useful for analyzing binary files for embedded credentials, URLs, API keys, etc.

End Result for Exercise 1:

Microcode value extracted successfully: 0xffffffff

Exercise 2: Basic cURL Queries & Data Extraction
Objective:

Learn how to use cURL for enumeration, downloading files, customizing headers, uploading data, and understanding HTTP mechanics.

1. Accessing a Web Server with cURL
Command
curl 192.168.177.200

Explanation

Performs a simple GET request.

Server returns HTML of default webpage.

Pentest relevance:
Baseline enumeration of HTTP services.

2. Download a File with Progress Bar
sudo curl URL -o filename


-o saves output to a file.

3. Create a 10MB Dummy File
dd if=/dev/zero of=testfile_10MB bs=10485760 count=1


Explanation:

dd creates arbitrary files for upload/download testing.

4. Verify and Copy File to Web Root
ls -lart /var/www/index.html
cp testfile_10MB /var/www/

5. Download the Test File from Web Server
sudo curl 192.168.177.200/testfile_10MB -o testfile.bin


Shows cURL's role in performing controlled transfer tests.

6. Verbose Mode for Request/Response Inspection
curl -v http://192.168.177.200


Displays:

Request headers (>)

Response headers (<)

cURL connection info (*)

7. Silent Modes
Hide body only
curl -svo /dev/null URL

Hide progress bar but show errors
curl -sSvo /dev/null URL

8. Add Custom Headers
Add header
curl -H 'X-My-Header-Test: CLIENTZZZ' http://192.168.177.200

Modify User-Agent
curl -H 'User-Agent: TEST' http://192.168.177.200


Pentest relevance:
Essential for evading WAF detection, simulating browsers, or API testing.

9. Sending POST Requests
Simple POST
curl --data "firstname=boolean&lastname=world" https://httpbin.org/post

URL-encoded POST
curl --data-urlencode "email=test@example.com" https://httpbin.org/post

File Upload
echo "This is a test file..." > file.txt
curl -F file=@file.txt https://httpbin.org/post

10. HEAD Request (metadata only)
sudo curl -I 192.168.177.200


Shows headers without downloading content.

End Result for Exercise 2:

cURL version identified successfully: 7.58.0

Exercise 3: Log Analysis and Manipulation with BASH
Objective:

Monitor logs, correlate network activity, identify attacker artifacts, and modify logs safely.

1. Live System Log Monitoring
tail -f /var/log/syslog


Shows real-time kernel and system messages.

2. Running a Nikto Web Scan
nikto -h 192.168.177.200


Purpose:
Simulate hostile traffic to generate log entries.

3. Monitor Apache Access Logs
cd /var/log/apache2
tail -f access.log


Observes:

IP addresses

Requested paths

User-Agent strings (e.g., Nikto)

4. Search Log for Specific Activity
cat access.log | grep index.php | more


Used for reviewing potential attack indicators or suspicious behavior.

5. Kernel-Level Network Logs
dmesg | grep network


Extracts kernel messages relating to network activity.

6. Authentication Log Monitoring
tail -f /var/log/auth.log


Triggers failure logs using:

ssh root@192.168.177.200


Extract failed login attempts:

awk '/sshd.*Failed/ { print $9 }' /var/log/auth.log

7. Log Manipulation (Cleaning IP Evidence)
Remove attacker IP
cat /var/log/apache2/access.log | grep -v "192.168.177.82" > cleaned.log

Verify absence
cat cleaned.log | grep "192.168.177.82"


Output: No results → logs successfully sanitized.

Pentest relevance:
Demonstrates how attackers cover tracks (for learning only—never in real engagements without permission).

8. BASH Socket Communication

A simple BASH one-liner using /dev/tcp/...
(Adjustment depends on OS version.)

End Result for Exercise 3:

Apache version successfully identified: 2.2.14










