'''

# 功能说明 ： 计算芯片处理延时（适用于SD5182H芯片）
# 输入参数 ：
 #           framesize  : 报文帧长（含FCS），单位：字节
 #			forward_mode   : 转发模式，可选值： bypass | pas，默认：bypass
 #			igr_port_mode  : 入端口模式，可选值： eth | pon（暂不支持），默认：eth
 #			igr_eth_mode   : 入端口ETH模式，可选值： copper | fiber，默认：copper
 #			egr_port_mode  : 出端口模式，可选值： eth | pon（暂不支持），默认：eth
 #			egr_eth_mode   : 出端口ETH模式，可选值： copper | fiber，默认：copper
 #			igr_eth_speed  : ETH入端口速率，单位： Gbps
 #			egr_eth_speed  : ETH出端口速率，单位： Gbps
 #			freq_dp    : IPS模块工作频率，单位：MHz，可选值：250 | 300，默认：250
 #			freq_ips   : IPS模块工作频率，单位：MHz，可选值：250 | 300，默认：250
 #			freq_ha    : HA模块工作频率，单位：MHz，固定值：400MHz
 #			freq_pqm   : PQM模块工作频率，单位：MHz，可选值：250 | 300，默认：250
 #			freq_pe    : PE模块工作频率，单位：MHz，可选值：250 | 300，默认：250
 #			freq_eqm   : EQM模块工作频率，单位：MHz，可选值：250 | 300，默认：250
 #			freq_eps   : EPS模块工作频率，单位：MHz，可选值：250 | 300，默认：250
 #			freq_psa   : PSA模块工作频率，单位：MHz，可选值：400 | 500，默认：500
'''
import getopt
import sys



opcode = 'write'
byte = 4096
pcie = "1.0"
lane = 1
mps = 128
ack = 5
fc = 4
header = 12
rcb = 64
mrrs = 256
latency = 300

def usage():
    print(
"""
Usage:sys.args[0] [option]
-p or --opcode: 操作类型，可选值： write | read，默认：write"
-p or --pcie: PCIE版本，可选值 1.0 | 2.0 | 3.0 | 4.0，默认值：1
-l or --line: PCIE Line数，可选值： 1~16，默认值：1"
-h or --header:TLP Header长度，可选值： 12 | 16，默认：12
-r or --rcb: Read Completion Boundary，可选值： 64 | 128，默认：64
-a or --ack: 接收多个TLP后响应ACK TLP数，默认：5
-f or --fc: 接收多个TLP后响应FC TLP数，默认：4
-t or --tency :latency 接收端接收到读请求到产生RCB的时间，单位ns，默认：300 

"""
    )
if len(sys.argv) != 21:
    usage()
    # sys.exit()
print(sys.argv)

try:
    opts, args = getopt.getopt(sys.argv[1:], "o:p:l:m:a:f:h:r:m:t:z", ["opcode=", "pcie=", "lane=", "maps=", "ack=", "fc=", "header=" "rcb=" "mrrs=" "tency=" "zbb="])
    print(opts)
except getopt.GetoptError:
    print("argv error,please input")


for cmd, arg in opts:

    if cmd in ("-o", "--opcode"):
        opcode = arg
        print(opcode)

    elif cmd in ("-p", "--pcie"):
        pcie = arg
        print(pcie)
    elif cmd in ("-l", "--lane"):
        lane = int(arg)
        print(lane)
    elif cmd in ("-m", "--maps"):
        maps = int(arg)
        print(maps)
    elif cmd in ("-a", "--ack"):
        ack = int(arg)
        print(ack)
    elif cmd in ("-f", "--fc"):
        fc = int(arg)
        print(fc)
    elif cmd in ("-h", "--header"):
        header = int(arg)
        print(header)
    elif cmd in ("-r", "--rcb"):
        rcb = int(arg)
        print(rcb)

    elif cmd in ("-t", "--tency"):
        latency = int(arg)
        print(latency)
    elif cmd in ("-z", "--zbb"):
        zbb=arg






if pcie == "1.0":
    gtps = 2.5
    code = 8.0 / 10
    start = 1
    sequence = 2
    ecrc = 4
    lcrc = 4
    end = 1

elif pcie == "2.0":
    print(lane)
    gtps = 5.0
    code = 8.0 / 10
    start = 1
    sequence = 2
    ecrc = 4
    lcrc = 4
    end = 1

elif pcie == "3.0":
    gtps = 8.0
    code = 128.0 / 130
    start = 4
    sequence = 2
    ecrc = 4
    lcrc = 4
    end = 0


elif pcie == "4.0":
    gtps = 16.0
    code = 128.0 / 130
    start = 4
    sequence = 2
    ecrc = 4
    lcrc = 4
    end = 0

elif pcie == "5.0":
    gtps = 32.0
    code = 128.0 / 130
    start = 4
    sequence = 2
    ecrc = 4
    lcrc = 4
    end = 0
overhead = start + sequence + ecrc + lcrc + end + header


print("----------------------------")
print("PCIE工作参数:")
print("----------------------------")
print("PCIE Version:", pcie)
print("PCIE Lane:", lane)
print("PCIE Maps:", mps)
print("OverHead Len:", overhead)
print("PCIE ACK:", ack)
print("PCIE FC:", fc)


def tput_pcie_write(mps, gtps, code, lane, overhead, ack, fc):

    print("----------------------")
    print("PCIE写操作吞吐量（单位：Gbps）")
    print("-----------------------")
    tput_by_latency = round((mps * gtps * code / ((mps + overhead) / lane + 1 / ack + 1 / fc)), 3)
    print("基于处理延时分析：", tput_by_latency)
    tput = round(1.0 * mps / (mps + overhead) * (gtps * code) * lane, 3)
    print("基于通道利用率：", tput)
    return


def tput_pcie_read(mps, gtps, code, lane, overhead, ack, fc, mrrs, rcb, latency):
    print("----------------------")
    print("PCIE读操作吞吐量（单位：Gbps）")
    print("-----------------------")
    tput_by_latency = round(
        8.0 * mrrs / ((1.0 * overhead / lane + 1.0 * (1 + mrrs) / rcb * (1.0 / ack + 1.0 / fc) + 1.0
                       * mrrs * (rcb + overhead) / (rcb * lane)) * 8.0 / (gtps * code) + latency), 3)
    print("基于处理延时分析：", tput_by_latency)
    return


if opcode == "write":

    tput_pcie_write(mps, gtps, code, lane, overhead, ack, fc)


elif opcode == "read":
    tput_pcie_read(mps, gtps, code, lane, overhead, ack, fc, mrrs, rcb, latency)

