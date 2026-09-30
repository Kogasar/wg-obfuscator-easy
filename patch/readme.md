Сборка докер образа из корня репозитория командой:
docker build -f patch/Dockerfile -t wg-obfuscator-easy:custom .

Внесены следующие изменения:
1. Файл backend/app/obfuscator/config.py

72я строка:
lines.append(f"fwmark = 0xdead")  # Add patch line

2. Файл backend/app/wireguard/config.py

53я строка:
lines.append("FwMark = 0xdead")  # Add patch line

66-69 строки:
            f"iptables -t nat -A POSTROUTING -o {config['wan_interface']} -j MASQUERADE; " # Edit patch line
            f"iptables -t mangle -A PREROUTING -i {config['wan_interface']} "  # Add patch line
            f"-p tcp --dport 5000 -j CONNMARK --set-mark 0xdead; "  # Add patch line
            f"iptables -t mangle -A OUTPUT -p tcp --sport 5000 "  # Add patch line
            f"-m connmark ! --mark 0 -j CONNMARK --restore-mark"  # Add patch line

75-78 строки:
            f"iptables -t nat -D POSTROUTING -o {config['wan_interface']} -j MASQUERADE; " # Edit patch line
            f"iptables -t mangle -D PREROUTING -i {config['wan_interface']} "  # Add patch line
            f"-p tcp --dport 5000 -j CONNMARK --set-mark 0xdead; "  # Add patch line
            f"iptables -t mangle -D OUTPUT -p tcp --sport 5000 "  # Add patch line
            f"-m connmark ! --mark 0 -j CONNMARK --restore-mark"  # Add patch line

93-97 строки:
            #lines.append(f"AllowedIPs = {config['subnet']}.{client['ip']}/32")  # Del patch line
            if client_name == "EXT_Server":  # Add patch line
                lines.append("AllowedIPs = 0.0.0.0/0") # Add patch line
            else: # Add patch line
                lines.append(f"AllowedIPs = {config['subnet']}.{client['ip']}/32") # Add patch line

3. В контейнер добавлен измененный файл /usr/bin/wg-quick
в котором изменена 240ая строка
с
[[ $proto == -4 ]] && cmd sysctl -q net.ipv4.conf.all.src_valid_mark=1
на
[[ $proto == -4 ]] && [[ $(sysctl -n net.ipv4.conf.all.src_valid_mark) != 1 ]] && cmd sysctl -q net.ipv4.conf.all.src_valid_mark=1
