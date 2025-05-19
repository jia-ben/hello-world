# IAP

## IAP ���

<details open><summary>���</summary>

![IAP ��������](image.png)

### ����APP��ʼ��ַFLASH

- IROM1
  - START: 0x0801,0000
  - SIZE: 0x0070,0000
  - ��ΪFLASH����ʼ��ַΪ0x0800,0000.��СΪ512K.64KΪiap
  - ƫ����Ӧ��Ϊ0x200�ı���

### app��ʼ��ַSRAM

- IROM1
  - START: 0x2000,1000
  - SEZE: 0xC000(48K)
- IRAM1: 0x2000,D000 + 0x3000
  - ��ΪSRAM��С64K,0-4K:Bootloader. 48K:APP. 12K:app�ڴ�
  - ƫ����Ӧ��Ϊ0x200�ı���

### �ж��������ƫ��������

`SCB->VTOR = FLASH_BASE | VECT_TAB_OFFSET;`
VTOR�Ĵ��������������ʼ��ַ

### app����

ʹ��iap������Ҫ.bin�ļ�,��keil/user����

#### �ܽ�

![APP��������](image-1.png)
</details>

## ������

1. ����app����
2. �洢��app����λ��
3. ����app

# USB ����

## ���

<details open> <summary> ���</summary>

USB-ͨ�ô�������

- �ĸ���:
  - VCC GND D+  D-
  - ��ֵ�ѹ����,
  - ����:15K��������
  - �豸:1.5K���赽VCC
    - ����:D+,����:D-,(�豸ʶ��)
- ������
  - PC�����������һ��ר�õ����ݻ�����
  - ��������С�� �˵���Ŀ,ÿ���˵�������ݷ��� ����
  - ����ĸ�ʽ��Ӳ�����
  - ������������:�������ĵ�ַ,��С,�ֽ���
  - USBģ��ͨ��16λ�Ĵ���-->������
- �ж�ӳ��
  - ������ͬ��NVIC��
  - 1.�����ȼ�20�� ����USB��ȷ�¼�
  - 2. �����ȼ�19��ͬ����˫������������ ��ȷ�¼�
  - 3.����42�� �����¼�

![USB�豸��ͼ](image-2.png)

</details>

# USB

[USB����˵��](https://wiki.stmicroelectronics.cn/stm32mcu/wiki/Introduction_to_USB_with_STM32)

# USB��������

## USBH_Process����

![Usr_cb](image-7.png)
![Class_cb](image-8.png)
![class_cb](image-9.png)
![usr_cb](image-10.png)
swtich phost->gState
״̬����

- HOST_IDLE
  - ������ResetPort()
  - �л�״̬��->HOST_IDLE
- HOST_WAIT_PRT_ENABLED
  - �ж϶˿�ʹ�ܣ��л�״̬��
- HOST_DEV_ATTACHED
  - ���� usr_cb-> DeviceAttached()
  - �ж����ö˿�==0��
  - usr_cb->ResetDevice()
  - ��ͨ��chaannel
  - �л�״̬��
- HOST_ENUMERATION
  - ����ö��()������ok
  - ����ö��()
  - �л�״̬��
- HOST_USR_INPUT
  - usr_cb->UserInput()������ok
  - class_cb->Init()
  - �л�״̬
- HOST_CLASS_REQUEST
  - class_cb->Requests()
  - ����ok�л�
  - ���ش���->������USBH_ErrorHandle()
  - >������:���ݴ����,ִ�в���,���л���״̬����error
- HOST_CLASS
  - class_cb->Machine()
  - ������USBH_ErrorHandle()

## USBH_MSC_Handle()

״̬��:.MSCState

---
