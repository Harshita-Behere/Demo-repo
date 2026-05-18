# Demo-repo

1. Get access to Windows intranet PC

2. Check/install:
   - Python
   - Git
   - Nmap
   - VS Code

3. Clone GitHub repository

4. Create project folders:
   backend/
   frontend/
   database/
   docs/
   sample_scans/

5. Create .gitignore file

6. Add to .gitignore:
   sample_scans/
   *.xml

7. Run:
   ipconfig

8. Find:
   - IPv4 address
   - subnet mask
   - subnet range

9. Run first scan:
   nmap -sn <subnet>

10. Verify scan shows:
    - IP
    - MAC
    - vendor
    - hostname

11. Save XML scan:
    nmap -sn <subnet> -oX sample_scans/scan1.xml

12. Run second scan later:
    nmap -sn <subnet> -oX sample_scans/scan2.xml

13. Open XML files and study structure

14. Push only code/project structure to GitHub

15. At home:
    - pull latest repo
    - start backend/parser development
