# **STM32WB Series BLE Template**

***

## Introduction

This application is an example that you can use as a starting point for developing your own BLE application, using the hardware included on the NUCLEO-WB55RG board.

## Setup

This application runs on a **NUCLEO-WB55RG board**.

A smartphone is needed to connect to STM32WB device with ST BLE Toolbox application. You can get it from your platform application store.  
After flashing your board you have to unplug jumpers present on JP5 in order to reach best current measurements. These jumpers connect STM32WB to ST-LINK debugger.  

To measure current consumption of the MCU you can use our X-NUCLEO-LPM01A board and power Nucleo board from JP2 connector.

## Application description

In order to make the program work, you must do the following:

- Open your toolchain
- Rebuild all files and flash the board with the executable file

On the Android/iOS device, enable the Bluetooth communications, and if not done before :

- Install the ST BLE Toolbox application :
   <https://wiki.st.com/stm32mcu/wiki/Connectivity:BLE_smartphone_applications#ST_BLE_Toolbox>
- Enable Bluetooth communications
- Open ST BLE Toolbox and Start Scanning
- Connect to MyCST application to access all the services and associated characteristics
- A display terminal can be used to display application messages

STM32WB is configured as a GATT server and supports :

- Advertising, restart advertising when disconnection occurs
- One service based on P2P Server (STM proprietary) : **My_Custom_Service**
- With one characteristic : **My_GPIO_Char** with write,read and notify property
- Writing to My_GPIO_Char toggles blue LED :
  - 0x00 LED off
  - 0x01 LED on
- Blue LED state readable by the client
- Push SW1 to notify
- Push SW4 to reset

For more details refer to the Application Note :  [ AN5289 - Building a Wireless application](https://www.st.com/resource/en/application_note/an5289-how-to-build-wireless-applications-with-stm32wb-mcus-stmicroelectronics.pdf)

## User code template : user sections

In order to help user to find where common codes are entered and then to modify it, this application come with a full project files template where we added minimum user code under several user sections. In fact, the generated source code contains several sections called **User Sections** where users can add custom application code parts. These sections are not erased/modified at project regeneration from CubeMX.

Here is a list of places where you can find the most useful user sections to modify to suit your own application. We also described how we achieved to enable the template application describes before.

- In **app_conf.h** :

Define the task for the characteristic **My_GPIO_Char** of **My_Custom_Service** by adding code in **CFG_Task_Id_With_HCI_Cmd_t** User Code Section:

```c++
/**< Add in that list all tasks that may send a ACI/HCI command */
typedef enum
{
  CFG_TASK_ADV_CANCEL_ID,
#if (L2CAP_REQUEST_NEW_CONN_PARAM != 0 )
  CFG_TASK_CONN_UPDATE_REG_ID,
#endif
  CFG_TASK_HCI_ASYNCH_EVT_ID,
  /* USER CODE BEGIN CFG_Task_Id_With_HCI_Cmd_t */
  CFG_TASK_SW1_BUTTON_PUSHED_ID,
  /* USER CODE END CFG_Task_Id_With_HCI_Cmd_t */
  CFG_LAST_TASK_ID_WITH_HCICMD,                                               /**< Shall be LAST in the list */
} CFG_Task_Id_With_HCI_Cmd_t;
```

- In **app_entry.c** :

Define the callback for handling the interrupts from SW1 presses by adding code in **FD_WRAP_FUNCTIONS** User Code Section:

```c++
/* USER CODE BEGIN FD_WRAP_FUNCTIONS */
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
  switch (GPIO_Pin)
  {
    case SW1_User_Pin:
      APP_BLE_Key_Button1_Action();
      break;
    default:
      break;
  }
  return;
}
/* USER CODE END FD_WRAP_FUNCTIONS */
```

- In **app_ble.c** :

Define function for SW1 action by adding code in **FD_SPECIFIC_FUNCTIONS** User Code Section:

```c++
/* USER CODE BEGIN FD_SPECIFIC_FUNCTIONS */
void APP_BLE_Key_Button1_Action(void)
{
  SW1_Button_Action();
}
/* USER CODE END FD_SPECIFIC_FUNCTIONS */
```

- In **app_ble.h** :

Declare the APP_BLE_Key_Button1_Action() function, in **EF** User Code Section:

```c++
/* Exported functions ---------------------------------------------*/
void APP_BLE_Init(void);
APP_BLE_ConnStatus_t APP_BLE_Get_Server_Connection_Status(void);

/* USER CODE BEGIN EF */
void APP_BLE_Key_Button1_Action(void);
/* USER CODE END EF */
```

- In **custom_app.h** :

Declare the SW1_Button_Action function, in **EF** User Code Section:

```c++
/* Exported functions ---------------------------------------------*/
void Custom_APP_Init(void);
void Custom_APP_Notification(Custom_App_ConnHandle_Not_evt_t *pNotification);
/* USER CODE BEGIN EF */
void SW1_Button_Action(void);
/* USER CODE END EF */
```

- In **custom_app.c** :

Add SW1_Status variable in  **CUSTOM_APP_Context_t** User Code Section :

```c++
/* Private typedef -----------------------------------------------------------*/
typedef struct
{
  /* My_Custom_Service */
  uint8_t               Gpio_c_Notification_Status;
  /* USER CODE BEGIN CUSTOM_APP_Context_t */
  uint8_t               SW1_Status;
  /* USER CODE END CUSTOM_APP_Context_t */

  uint16_t              ConnectionHandle;
} Custom_App_Context_t;

/* USER CODE BEGIN PTD */

/* USER CODE END PTD */
```

In the same file, update snippets shown. The following traces code is to indicate the Smartphone requests to enable or disable notifications and write requests on My_Custom_Service service. 

In **CUSTOM_STM_GPIO_C_WRITE_NO_RESP_EVT** User Code Section :

```c++
/* Functions Definition ------------------------------------------------------*/
void Custom_STM_App_Notification(Custom_STM_App_Notification_evt_t *pNotification)
{
  /* USER CODE BEGIN CUSTOM_STM_App_Notification_1 */

  /* USER CODE END CUSTOM_STM_App_Notification_1 */
  switch (pNotification->Custom_Evt_Opcode)
  {
    /* USER CODE BEGIN CUSTOM_STM_App_Notification_Custom_Evt_Opcode */

    /* USER CODE END CUSTOM_STM_App_Notification_Custom_Evt_Opcode */

    /* My_Custom_Service */
    case CUSTOM_STM_GPIO_C_READ_EVT:
      /* USER CODE BEGIN CUSTOM_STM_GPIO_C_READ_EVT */

      /* USER CODE END CUSTOM_STM_GPIO_C_READ_EVT */
      break;

    case CUSTOM_STM_GPIO_C_WRITE_NO_RESP_EVT:
      /* USER CODE BEGIN CUSTOM_STM_GPIO_C_WRITE_NO_RESP_EVT */
        APP_DBG_MSG("\r\n\r** CUSTOM_STM_GPIO_C_WRITE_NO_RESP_EVT \n");
        APP_DBG_MSG("\r\n\r** Write Data: 0x%02X %02X \n", pNotification->DataTransfered.pPayload[0], pNotification->DataTransfered.pPayload[1]);
        if(pNotification->DataTransfered.pPayload[1] == 0x01)
        {
        	HAL_GPIO_WritePin(Blue_Led_GPIO_Port, Blue_Led_Pin, GPIO_PIN_SET);
        }
        if(pNotification->DataTransfered.pPayload[1] == 0x00)
        {
        	HAL_GPIO_WritePin(Blue_Led_GPIO_Port, Blue_Led_Pin, GPIO_PIN_RESET);
        }

      /* USER CODE END CUSTOM_STM_GPIO_C_WRITE_NO_RESP_EVT */
```

In **CUSTOM_STM_GPIO_C_NOTIFY_ENABLED_EVT** and **CUSTOM_STM_GPIO_C_NOTIFY_DISABLED_EVT** User Code Sections :

```c++
case CUSTOM_STM_GPIO_C_NOTIFY_ENABLED_EVT:
      /* USER CODE BEGIN CUSTOM_STM_GPIO_C_NOTIFY_ENABLED_EVT */
        APP_DBG_MSG("\r\n\r** CUSTOM_STM_GPIO_C_NOTIFY_ENABLED_EVT \n");

        Custom_App_Context.Gpio_c_Notification_Status = 1;

      /* USER CODE END CUSTOM_STM_GPIO_C_NOTIFY_ENABLED_EVT */
      break;

    case CUSTOM_STM_GPIO_C_NOTIFY_DISABLED_EVT:
      /* USER CODE BEGIN CUSTOM_STM_GPIO_C_NOTIFY_DISABLED_EVT */
        APP_DBG_MSG("\r\n\r** CUSTOM_STM_GPIO_C_NOTIFY_DISABLED_EVT \n");

        Custom_App_Context.Gpio_c_Notification_Status = 0;

      /* USER CODE END CUSTOM_STM_GPIO_C_NOTIFY_DISABLED_EVT */
      break;
```

Add code snippets in Custom_App_Init() function to register the send notification task that is started when SW1 button is pressed, in **CUSTOM_APP_Init** User Section:

```c++
void Custom_APP_Init(void)
{
  /* USER CODE BEGIN CUSTOM_APP_Init */
	  UTIL_SEQ_RegTask(1<< CFG_TASK_SW1_BUTTON_PUSHED_ID, UTIL_SEQ_RFU, Custom_Gpio_c_Send_Notification);
	  Custom_App_Context.Gpio_c_Notification_Status = 0;
	  Custom_App_Context.SW1_Status = 0;
  /* USER CODE END CUSTOM_APP_Init */
  return;
}
```

And code snippet to allow the task to run once SW1 is pressed, in **FD_LOCAL_FUNCTIONS** User Section:

```c++
/* USER CODE BEGIN FD_LOCAL_FUNCTIONS*/
void SW1_Button_Action(void)
{
  UTIL_SEQ_SetTask( 1<<CFG_TASK_SW1_BUTTON_PUSHED_ID, CFG_SCH_PRIO_0);

  return;
}
/* USER CODE END FD_LOCAL_FUNCTIONS*/
```

Add code snippet for sending notification request to the smart phone, in **Gpio_c_NS_1** User Section:

```c++
void Custom_Gpio_c_Send_Notification(void) /* Property Notification */
{
  uint8_t updateflag = 0;

  /* USER CODE BEGIN Gpio_c_NS_1*/
  if(Custom_App_Context.Gpio_c_Notification_Status)
   {
     Custom_STM_App_Update_Char(CUSTOM_STM_GPIO_C, (uint8_t *)NotifyCharData);
     if (Custom_App_Context.SW1_Status == 0)
     {
       Custom_App_Context.SW1_Status = 1;
       NotifyCharData[0] = 0x00;
       NotifyCharData[1] = 0x01;
     }
     else
     {
       Custom_App_Context.SW1_Status = 0;
       NotifyCharData[0] = 0x00;
       NotifyCharData[1] = 0x00;
     }

     APP_DBG_MSG("-- CUSTOM APPLICATION SERVER  : INFORM CLIENT BUTTON 1 PUSHED \n");
     APP_DBG_MSG(" \n\r");
     Custom_STM_App_Update_Char(CUSTOM_STM_GPIO_C, (uint8_t *)NotifyCharData);
   }
   else
   {
     APP_DBG_MSG("-- CUSTOM APPLICATION : CAN'T INFORM CLIENT -  NOTIFICATION DISABLED\n");
   }

  /* USER CODE END Gpio_c_NS_1*/

  if (updateflag != 0)
  {
    Custom_STM_App_Update_Char(CUSTOM_STM_GPIO_C, (uint8_t *)NotifyCharData);
  }

  /* USER CODE BEGIN Gpio_c_NS_Last*/

  /* USER CODE END Gpio_c_NS_Last*/

  return;
}
```

- In **custom_stm.c** :

Update Custom_STM_Event_Handler() function to manage My_GPIO_Char write, in **CUSTOM_STM_Service_1_Char_1_ACI_GATT_ATTRIBUTE_MODIFIED_VSEVT_CODE** User Section:

```c++
static SVCCTL_EvtAckStatus_t Custom_STM_Event_Handler(void *Event)
{
  […]
 event_pckt = (hci_event_pckt *)(((hci_uart_pckt*)Event)->data);

  switch(event_pckt->evt)
  {
    case HCI_VENDOR_SPECIFIC_DEBUG_EVT_CODE:
      blecore_evt = (evt_blecore_aci*)event_pckt->data;
      switch(blecore_evt->ecode)
      {
        case ACI_GATT_ATTRIBUTE_MODIFIED_VSEVT_CODE:
        […]
            else if (attribute_modified->Attr_Handle == (CustomContext.CustomGpio_CHdle + CHARACTERISTIC_VALUE_ATTRIBUTE_OFFSET))
          {
            return_value = SVCCTL_EvtAckFlowEnable;
            /* USER CODE BEGIN CUSTOM_STM_Service_1_Char_1_ACI_GATT_ATTRIBUTE_MODIFIED_VSEVT_CODE */

            Notification.Custom_Evt_Opcode = CUSTOM_STM_GPIO_C_WRITE_NO_RESP_EVT;
            Notification.DataTransfered.Length=attribute_modified->Attr_Data_Length;
            Notification.DataTransfered.pPayload=attribute_modified->Attr_Data;
            Custom_STM_App_Notification(&Notification);

            /* USER CODE END CUSTOM_STM_Service_1_Char_1_ACI_GATT_ATTRIBUTE_MODIFIED_VSEVT_CODE */
          } /* if (attribute_modified->Attr_Handle == (CustomContext.CustomGpio_CHdle + CHARACTERISTIC_VALUE_ATTRIBUTE_OFFSET))*/
          /* USER CODE BEGIN EVT_BLUE_GATT_ATTRIBUTE_MODIFIED_END */

          /* USER CODE END EVT_BLUE_GATT_ATTRIBUTE_MODIFIED_END */
          break;
```

## Troubleshooting

**Caution** : Issues and the pull-requests are **not supported** to submit problems or suggestions related to the software delivered in this repository. The STM32WB-BLE-Template example is being delivered as-is, and not necessarily supported by ST.

**For any other question** related to the product, the hardware performance or characteristics, the tools, the environment, you can submit it to the **ST Community** on the STM32 MCUs related [page](https://community.st.com/s/topic/0TO0X000000BSqSWAW/stm32-mcus).


[def]: https://