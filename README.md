# 0xlicker-skull-1
import requests
import socket
import datetime
import sys
import urllib3
import time
from concurrent.futures import ThreadPoolExecutor
from colorama import init, Fore, Style

# تعطيل تحذيرات SSL
urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
init(autoreset=True)

# اللوقو: الجمجمة مسحوبة لليسار والنص مقابلها مباشرة
LOGO = f"""{Fore.RED}
  .......','.                                                   
 .':looodxk0KOOko:'                ▄▄▄▄▄          ▄▄                                  ▄▄ ▄▄ 
.:oxkO000KXXKOO000kl'.           ▄███████▄        ██ ▀▀          ▄▄                ▄▄         ██ ██ 
.,::lx0KK0KK0OOO0Oxooo:.         ███   ███ ██ ██ ██ ██  ▄████ ██ ▄█▀ ▄█▀█▄ ████▄   ▄█▀▀▀ ██ ▄█▀ ██ ██ ██ ██ 
..','..o0OOKK0OOOOkxxdol:'       ███▄▄▄███  ███  ██ ██  ██    ████   ██▄█▀ ██ ▀▀   ▀███▄ ████   ██ ██ ██ ██ 
  .....ckdlc;'...;ldl'......      ▀█████▀  ██ ██ ██ ██▄ ▀████ ██ ▀█▄ ▀█▄▄▄ ██      ▄▄▄█▀ ██ ▀█▄ ▀██▀█ ██ ██ 
      .:;.      .,::c,.    .      
      ,'        .;l,.;c,. ...     {Fore.WHITE}[ 0xSUB: Advanced Security Engine - SQL & Backdoor Recon ]{Fore.RED}
     .;:.       .:c;..   .:c;     {Fore.WHITE}[ Deep Port Analysis: SQL (3306, 5432, 1433) & Remote (22, 23, 3389) ]{Fore.RED}
      .;;'...,,cxOxl.    ,ddc.    
      ..,oxdddoodOd.    ..''',.   
        .lOd;..';od'  ..... ...   
         ':,.':lcokxlll;,;'       
             .;lldkO0K0Oxd:.      
               .,cc:::;;,''.      
{Style.RESET_ALL}"""

def print_intro():
    print(LOGO)
    print("")

TARGET_PORTS = {"22": "SSH", "23": "Telnet", "3306": "SQL", "5432": "SQL", "1433": "SQL", "3389": "RDP"}

def scan_ports(ip):
    found = []
    for port, name in TARGET_PORTS.items():
        try:
            sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            sock.settimeout(0.2)
            if sock.connect_ex((ip, int(port))) == 0:
                found.append(f"{name}({port})")
            sock.close()
        except: pass
    return ",".join(found) if found else "None"

def get_status(url):
    now = f"{Fore.LIGHTGREEN_EX}{datetime.datetime.now().strftime('%H:%M:%S')}{Style.RESET_ALL}"
    target_clean = url.replace('https://', '').replace('http://', '').strip('/')
    box = f"[{Fore.WHITE}▒{Fore.RED}▒{Fore.WHITE}▒{Style.RESET_ALL}]"

    try:
        ip = socket.gethostbyname(target_clean)
        ports = scan_ports(ip)
        res = requests.get(url, timeout=4, verify=False, allow_redirects=True)
        
        # تنسيق الحالة 200 OK بالأخضر
        status_text = "200 OK" if res.status_code == 200 else str(res.status_code)
        status_color = Fore.LIGHTGREEN_EX if res.status_code == 200 else Fore.YELLOW
        status_final = f"{status_color}{status_text:<8}{Style.RESET_ALL}"
        
        # التقدير المختصر والموزون
        waf = any(s in str(res.headers).lower() for s in ['cloudflare', 'akamai', 'aliyun', 'fortinet', 'incapsula', 'sucuri'])
        if waf: level = f"[{Fore.RED}HARD{Style.RESET_ALL}]"
        elif 'X-Frame-Options' in res.headers or 'Content-Security-Policy' in res.headers: level = f"[{Fore.LIGHTYELLOW_EX}MID {Style.RESET_ALL}]"
        else: level = f"[{Fore.GREEN}LOW {Style.RESET_ALL}]"

        # نظام المسافات لضمان التساوي (المسطرة)
        print(f"{box} > [{now}] > [{Fore.WHITE}{ip:<16}{Style.RESET_ALL}] > [{Fore.CYAN}{target_clean:<40}{Style.RESET_ALL}] > {level} > {status_final} > [Ports:{Fore.YELLOW}{ports}{Style.RESET_ALL}]")
    except:
        pass

def start_scan(file_path):
    print_intro()
    try:
        with open(file_path, 'r') as f:
            targets = list(set([line.strip() for line in f if line.strip()]))
        with ThreadPoolExecutor(max_workers=30) as executor:
            for t in targets: executor.submit(get_status, f"https://{t}")
    except: pass

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print_intro()
        print(f"{Fore.RED}Usage: python3 suba.py targets.txt{Style.RESET_ALL}")
        sys.exit()
    start_scan(sys.argv[1])
