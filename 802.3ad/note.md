# 在没插网线或者万兆线路时，需要对万兆网卡进行bond配置。记录，未测试。
# 查找所有万兆网卡
bonded_interfaces=()
for iface in $(ls /sys/class/net); do
  # 获取网卡支持的链路模式
  supported_modes=$(ethtool "$iface" 2>/dev/null | grep "Supported link modes" -A 1 | tail -n 1)
  
  # 检查是否支持万兆模式
  if echo "$supported_modes" | grep -q "10000base"; then
    bonded_interfaces+=("$iface")
  fi
done

# 检查是否找到至少两个万兆网卡
INTERFACE_COUNT=${#bonded_interfaces[@]}
if [[ $INTERFACE_COUNT -lt 2 ]]; then
  echo "Error: Less than two 10GbE interfaces found. Found $INTERFACE_COUNT" >> /tmp/post_install.log
  exit 1
fi

# 将找到的网卡赋值给变量，最多取前两个
INTERFACE1="${bonded_interfaces[0]}"
INTERFACE2="${bonded_interfaces[1]}"
echo "Interface 1: $INTERFACE1"
echo "Interface 2: $INTERFACE2"

