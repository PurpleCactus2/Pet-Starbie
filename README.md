# Starbie
A motion controlled PCB pet!

<img width="751" height="564" alt="PCB Render" src="https://github.com/user-attachments/assets/b47fb055-e034-4edb-ae34-a0e7c0f3f96a" />

### Introduction
This is Starbie, a motion controlled, Tamagotchi style digital pet meant to keep you company on your desk. It uses a microcontroller, motion and enviroment sensor, along with 2 buttons to allow you to interact with it. It was made with the help of this wonderful [guide](https://github.com/SharKingStudios/Starbie/tree/main) by Logan Peterson which guided me through the entire process of making my first PCB.

Its main functions include:
- A stats screen for your pet
- Nap, Play and Pet actions
- A shake interaction

And you can also customise your own sprite!

However, I have also added a couple of my own features:
- A basic power LED indicating when starbie is on
- An external battery circuit making starbie portable
- A 3D printed enclosure to tie everything together

### PCB and Schematic
This is the entire schematic using the connections and components that are described in the guide.
The ESP32-C3 connects the buttons through 2 GPIO pins, the screen and the motion sensor through I2C, and also supplies the steady 3.3V that the circuit needs.

<img width="1027" height="540" alt="Schematic Screenshot" src="https://github.com/user-attachments/assets/aa34d93c-659d-4542-b5a1-ef6acc125ab9" />
<br>
<br>
Here are the 2 things that I added:
<br>
<br>
This is the basic power LED which connects to the 3.3V output of the ESP32-C3 through a 220Ω resistor. The LED itself will just be a standard 5mm 2V red LED that indicates whenever the circuit is connected to power.
<br>
<img width="374" height="714" alt="Power LED" src="https://github.com/user-attachments/assets/d5361e27-d755-443f-b2e6-0e1faa57f43e" />
<br>
<br>
This is the external battery circuit that will power starbie through a battery. It will use a 500mAh 3.7V 503035 liPo that connects straight to the battery pads on the underside of the Xiao, meaning it will be charged and regulated with the battery charger already integrated into the microcontroller. A switch has also been included in line from the battery so that starbie can be turned off easily.
<br>
<img width="1023" height="452" alt="Battery Circuit" src="https://github.com/user-attachments/assets/da613b3e-346b-40da-9d34-13edfc7361f7" />
<br>
<br>
PCB:
<br>
<br>
Here, I have used a similar layout and shape as shown in the guide. <br> <br>
At first, I tried experimenting with the positioning of the buttons by having them side by side, but this turned out to make to routing more complicated by requiring more traces on the bottom layer. Because of this, I decided to align the buttons vertically, even if it is a little more inconvenient. I have also added a couple of M3 screw holes on the edges so that I can mount the PCB in the enclosure that I designed.

<img width="975" height="844" alt="PCB Screenshot" src="https://github.com/user-attachments/assets/ecf05359-d4f9-4bc2-84c7-a0eec92e6835" />
<br>
<br>
Here are the contacts for the battery pads of the Xiao:

<img width="668" height="791" alt="Battery Pads PCB" src="https://github.com/user-attachments/assets/2bcabdc7-d7fc-4982-b4c0-b12390785459" />

These pads won't actually be making contact with the pads on the Xiao, since it will be raised on top of header pins. But, I have put through-holes on these pads to allow for header pins to be pushed through them. This should allow the battery to be soldered directly to these pins from the bottom, making managing space a lot easier.

### Code
Most of the code is from the preset given by the guide, but there are some parts which I have modified:
<br>
<br>
These 2 pins were previously mismatched with SCL on 7 and SDA on 6 while my PCB and schematic has SCL on 6 and SDA on 7. This would have thrown an error where the Xiao would not be able to communicate with the screen or motion sensor. Luckily the ESP32-C3 is able to reroute these I2C connections easily.
~~~cpp
const int I2C_SDA_PIN = 7;
const int I2C_SCL_PIN = 6;
~~~
<br>
<br>
This was an optional function given in the guide that could be inserted above the drawHeart function. It will create sparkles behind the pet as it moves.

~~~cpp
void drawSparkle(int x, int y) {
  display.drawPixel(x, y - 2, SSD1306_WHITE);
  display.drawPixel(x - 1, y - 1, SSD1306_WHITE);
  display.drawPixel(x + 1, y - 1, SSD1306_WHITE);
  display.drawPixel(x - 2, y, SSD1306_WHITE);
  display.drawPixel(x, y, SSD1306_WHITE);
  display.drawPixel(x + 2, y, SSD1306_WHITE);
  display.drawPixel(x - 1, y + 1, SSD1306_WHITE);
  display.drawPixel(x + 1, y + 1, SSD1306_WHITE);
  display.drawPixel(x, y + 2, SSD1306_WHITE);
}
~~~

This is what actually calls the drawSparkle function
~~~cpp
if (!isNapping() && (now / 250) % 2 == 0) {
  drawSparkle(petX - 4, petY + 18);
}
~~~
<br>
<br>
This is a byte list created from a custom 32x32px sprite which I drew.

~~~cpp
const uint8_t PROGMEM PET_SPRITE[] = {
  0x00, 0x0f, 0xf0, 0x00, 0x00, 0x70, 0x0e, 0x00, 0x01, 0x80, 0x01, 0x80, 0x02, 0x00, 0x00, 0x40, 
	0x04, 0x00, 0x00, 0x20, 0x08, 0x00, 0x00, 0x10, 0x10, 0x00, 0x00, 0x08, 0x20, 0x00, 0x38, 0x04, 
	0x20, 0x00, 0x38, 0x04, 0x40, 0x00, 0x30, 0x02, 0x40, 0x00, 0x00, 0x02, 0x40, 0x00, 0x00, 0x02, 
	0x80, 0x00, 0x00, 0x3c, 0x80, 0x00, 0x03, 0xe0, 0x80, 0x00, 0x1e, 0x00, 0x80, 0x00, 0x20, 0x00, 
	0x80, 0x00, 0x20, 0x00, 0x80, 0x00, 0x1e, 0x00, 0x80, 0x00, 0x03, 0xe0, 0x80, 0x00, 0x00, 0x3c, 
	0x40, 0x00, 0x00, 0x02, 0x40, 0x00, 0x00, 0x02, 0x40, 0x00, 0x00, 0x02, 0x20, 0x00, 0x00, 0x04, 
	0x20, 0x00, 0x00, 0x04, 0x10, 0x00, 0x00, 0x08, 0x08, 0x00, 0x00, 0x10, 0x04, 0x00, 0x00, 0x20, 
	0x02, 0x00, 0x00, 0x40, 0x01, 0x80, 0x01, 0x80, 0x00, 0x70, 0x0e, 0x00, 0x00, 0x0f, 0xf0, 0x00,
};
~~~

The sprite should look like this:

<img width="32" height="32" alt="Pac Man Sprite" src="https://github.com/user-attachments/assets/7605e1dd-0ee2-4dc2-abb2-5c2a01e3cbf6" />

### Enclosure
I wanted to make the PCB a little more personalised and I also wanted to have a neat place to store the battery. Because of this, I decided to design a 3D printable enclosure meant to be assembled with screws and a little bit of glue.

This is the enclosure as a whole. It has several holes on the top for each component that sticks through, along with 4 M2.5x8 screw holes along the edge of the lid for assembly.

<img width="1085" height="762" alt="CAD Screenshot" src="https://github.com/user-attachments/assets/261cf540-edd5-4df6-9f74-b336aeb3a1d0" />
<br>
<br>
<br>
This is how the battery and the switch are arranged below the PCB. The battery is completely hidden inside the bottom of the case while the switch sticks out a little bit through a hole so that you can use it.

<img width="839" height="700" alt="Bottom CAD screenshot" src="https://github.com/user-attachments/assets/8b23950f-5d51-4565-ae4a-2413339afe1a" />
<br>
<br>
<br>
This is how the entire enclosure is assembled. The battery and the PCB are inserted into the case first, along with 4 M3x8 that screw into holes in the bottom of the case. The lid is then put on with 4 M2.5x8 along its edges. Finally after trimming the pins of the switch, it is super-glued onto the back of the PCB next to the battery.

<img width="763" height="610" alt="Exploded View" src="https://github.com/user-attachments/assets/86b32f2f-18ba-4b32-922e-55aef99514db" />
<br>
<br>
<br>
Here is a more detailed view of the screw holes inside the case as well as of the battery and switch placement:

<img width="641" height="557" alt="Battery and Switch" src="https://github.com/user-attachments/assets/e9dc8ddb-39ac-46de-b902-cb4f2160892c" />

### BOM
| Part | What it's for | Qty | Unit | Total | Vendor |
| --- | --- | --- | --- | --- | --- |
| [XIAO ESP32 C3](https://www.aliexpress.com/item/1005008791937363.html?spm=a2g0o.productlist.main.57.73cf3205oRycWV&algo_pvid=b26d9f0b-0926-4227-8f6e-a914df534036&algo_exp_id=b26d9f0b-0926-4227-8f6e-a914df534036-54&pdp_ext_f=%7B%22order%22%3A%221488%22%2C%22spu_best_type%22%3A%22price%22%2C%22eval%22%3A%221%22%2C%22fromPage%22%3A%22search%22%7D&pdp_npi=6%40dis%21NOK%2183.14%2110.88%21%21%218.33%211.09%21%402103810f17916626052361400e0f0b%2112000052930246748%21sea%21NO%218463637778%21X%211%210%21n_tag%3A-29911%3Bd%3A22d43e35%3Bm03_new_user%3A-29895%3BpisId%3A5000000219435413&curPageLogUid=9fTnC7rBXexc&utparam-url=scene%3Asearch%7Cquery_from%3A%7Cx_object_id%3A1005008791937363%7C_p_origin_prod%3A) | The main microcontroller | 1 | $8.84 | $8.84 | [AliExpress](https://www.aliexpress.com/item/1005008791937363.html?spm=a2g0o.productlist.main.57.73cf3205oRycWV&algo_pvid=b26d9f0b-0926-4227-8f6e-a914df534036&algo_exp_id=b26d9f0b-0926-4227-8f6e-a914df534036-54&pdp_ext_f=%7B%22order%22%3A%221488%22%2C%22spu_best_type%22%3A%22price%22%2C%22eval%22%3A%221%22%2C%22fromPage%22%3A%22search%22%7D&pdp_npi=6%40dis%21NOK%2183.14%2110.88%21%21%218.33%211.09%21%402103810f17916626052361400e0f0b%2112000052930246748%21sea%21NO%218463637778%21X%211%210%21n_tag%3A-29911%3Bd%3A22d43e35%3Bm03_new_user%3A-29895%3BpisId%3A5000000219435413&curPageLogUid=9fTnC7rBXexc&utparam-url=scene%3Asearch%7Cquery_from%3A%7Cx_object_id%3A1005008791937363%7C_p_origin_prod%3A) |
| [0.96" OLED](https://www.aliexpress.com/item/1005006141235306.html?spm=a2g0o.productlist.main.1.4f9d12780KL1MO&algo_pvid=39c5fa34-e376-41e0-8849-d3f33eab2a7b&algo_exp_id=39c5fa34-e376-41e0-8849-d3f33eab2a7b-0&pdp_ext_f=%7B%22order%22%3A%2226478%22%2C%22spu_best_type%22%3A%22price%22%2C%22eval%22%3A%221%22%2C%22fromPage%22%3A%22search%22%7D&pdp_npi=6%40dis%21NOK%2122.70%2122.70%21%21%2115.22%2115.22%21%400b5e1a3817916636389462446e0d42%2112000035944225406%21sea%21NO%218463637778%21X%211%210%21n_tag%3A-29911%3Bd%3A22d43e35%3Bm03_new_user%3A-29895&curPageLogUid=3zuT5PcCui5q&utparam-url=scene%3Asearch%7Cquery_from%3A%7Cx_object_id%3A1005006141235306%7C_p_origin_prod%3A) | For displaying the sprite and actions | 1 | $2.31 | $2.31 | [AliExpress](https://www.aliexpress.com/item/1005006141235306.html?spm=a2g0o.productlist.main.1.4f9d12780KL1MO&algo_pvid=39c5fa34-e376-41e0-8849-d3f33eab2a7b&algo_exp_id=39c5fa34-e376-41e0-8849-d3f33eab2a7b-0&pdp_ext_f=%7B%22order%22%3A%2226478%22%2C%22spu_best_type%22%3A%22price%22%2C%22eval%22%3A%221%22%2C%22fromPage%22%3A%22search%22%7D&pdp_npi=6%40dis%21NOK%2122.70%2122.70%21%21%2115.22%2115.22%21%400b5e1a3817916636389462446e0d42%2112000035944225406%21sea%21NO%218463637778%21X%211%210%21n_tag%3A-29911%3Bd%3A22d43e35%3Bm03_new_user%3A-29895&curPageLogUid=3zuT5PcCui5q&utparam-url=scene%3Asearch%7Cquery_from%3A%7Cx_object_id%3A1005006141235306%7C_p_origin_prod%3A) |
| [MPU6050](https://www.aliexpress.com/item/1005010602044047.html?spm=a2g0o.productlist.main.1.48303357RDfjHx&algo_pvid=4210383c-c501-405f-a9e2-f9a703648051&algo_exp_id=4210383c-c501-405f-a9e2-f9a703648051-0&pdp_ext_f=%7B%22order%22%3A%22788%22%2C%22eval%22%3A%221%22%2C%22fromPage%22%3A%22search%22%7D&pdp_npi=6%40dis%21NOK%2120.03%2120.03%21%21%2113.43%2113.43%21%40210384a717916639655816378e0e45%2112000052945138544%21sea%21NO%218463637778%21X%211%210%21n_tag%3A-29911%3Bd%3A22d43e35%3Bm03_new_user%3A-29895&curPageLogUid=bgBBlxWptc6U&utparam-url=scene%3Asearch%7Cquery_from%3A%7Cx_object_id%3A1005010602044047%7C_p_origin_prod%3A) | For calculating  orientation and acceleration | 1 | $2.10 | $2.10 | [AliExpress](https://www.aliexpress.com/item/1005010602044047.html?spm=a2g0o.productlist.main.1.48303357RDfjHx&algo_pvid=4210383c-c501-405f-a9e2-f9a703648051&algo_exp_id=4210383c-c501-405f-a9e2-f9a703648051-0&pdp_ext_f=%7B%22order%22%3A%22788%22%2C%22eval%22%3A%221%22%2C%22fromPage%22%3A%22search%22%7D&pdp_npi=6%40dis%21NOK%2120.03%2120.03%21%21%2113.43%2113.43%21%40210384a717916639655816378e0e45%2112000052945138544%21sea%21NO%218463637778%21X%211%210%21n_tag%3A-29911%3Bd%3A22d43e35%3Bm03_new_user%3A-29895&curPageLogUid=bgBBlxWptc6U&utparam-url=scene%3Asearch%7Cquery_from%3A%7Cx_object_id%3A1005010602044047%7C_p_origin_prod%3A) |
| [DHT11](https://www.aliexpress.com/item/1005007728845619.html?spm=a2g0o.productlist.main.37.478071f3CCmwdm&algo_pvid=8a062715-841e-4e9b-bf6d-60feb66e7b31&algo_exp_id=8a062715-841e-4e9b-bf6d-60feb66e7b31-36&pdp_ext_f=%7B%22order%22%3A%222170%22%2C%22spu_best_type%22%3A%22price%22%2C%22eval%22%3A%221%22%2C%22fromPage%22%3A%22search%22%7D&pdp_npi=6%40dis%21NOK%2125.07%2116.96%21%21%2116.81%2111.37%21%4021038db317916647905926102e0fe8%2112000042055448713%21sea%21NO%218463637778%21X%211%210%21n_tag%3A-29911%3Bd%3A22d43e35%3Bm03_new_user%3A-29895&curPageLogUid=2wPSphHKUwT4&utparam-url=scene%3Asearch%7Cquery_from%3A%7Cx_object_id%3A1005007728845619%7C_p_origin_prod%3A) | For measuring temperature and humidity (Just the bare module, no breakout board) | 1 | $2.80 | $2.80 | [AliExpress](https://www.aliexpress.com/item/1005007728845619.html?spm=a2g0o.productlist.main.37.478071f3CCmwdm&algo_pvid=8a062715-841e-4e9b-bf6d-60feb66e7b31&algo_exp_id=8a062715-841e-4e9b-bf6d-60feb66e7b31-36&pdp_ext_f=%7B%22order%22%3A%222170%22%2C%22spu_best_type%22%3A%22price%22%2C%22eval%22%3A%221%22%2C%22fromPage%22%3A%22search%22%7D&pdp_npi=6%40dis%21NOK%2125.07%2116.96%21%21%2116.81%2111.37%21%4021038db317916647905926102e0fe8%2112000042055448713%21sea%21NO%218463637778%21X%211%210%21n_tag%3A-29911%3Bd%3A22d43e35%3Bm03_new_user%3A-29895&curPageLogUid=2wPSphHKUwT4&utparam-url=scene%3Asearch%7Cquery_from%3A%7Cx_object_id%3A1005007728845619%7C_p_origin_prod%3A) |
| [MX Switches](https://www.aliexpress.com/item/1005006376024657.html?spm=a2g0o.productlist.main.1.165735f5ClXEcQ&algo_pvid=4a830536-a4b2-499f-a21c-93d9edf09368&algo_exp_id=4a830536-a4b2-499f-a21c-93d9edf09368-0&pdp_ext_f=%7B%22order%22%3A%222317%22%2C%22eval%22%3A%221%22%2C%22fromPage%22%3A%22search%22%7D&pdp_npi=6%40dis%21NOK%2137.52%2110.88%21%21%2125.16%217.30%21%400b0fe40d17916655923207967e1029%2112000037159110206%21sea%21NO%218463637778%21X%211%210%21n_tag%3A-29911%3Bd%3A22d43e35%3Bm03_new_user%3A-29895%3BpisId%3A5000000219435413&curPageLogUid=OjmnomvXuXFE&utparam-url=scene%3Asearch%7Cquery_from%3A%7Cx_object_id%3A1005006376024657%7C_p_origin_prod%3A) | Buttons to control different functions (Minimum order size: 10pcs) | 1 | $3.93 | $3.93 | [AliExpress](https://www.aliexpress.com/item/1005006376024657.html?spm=a2g0o.productlist.main.1.165735f5ClXEcQ&algo_pvid=4a830536-a4b2-499f-a21c-93d9edf09368&algo_exp_id=4a830536-a4b2-499f-a21c-93d9edf09368-0&pdp_ext_f=%7B%22order%22%3A%222317%22%2C%22eval%22%3A%221%22%2C%22fromPage%22%3A%22search%22%7D&pdp_npi=6%40dis%21NOK%2137.52%2110.88%21%21%2125.16%217.30%21%400b0fe40d17916655923207967e1029%2112000037159110206%21sea%21NO%218463637778%21X%211%210%21n_tag%3A-29911%3Bd%3A22d43e35%3Bm03_new_user%3A-29895%3BpisId%3A5000000219435413&curPageLogUid=OjmnomvXuXFE&utparam-url=scene%3Asearch%7Cquery_from%3A%7Cx_object_id%3A1005006376024657%7C_p_origin_prod%3A) |
| [PCB](https://cart.jlcpcb.com/quote?spm=jlcpcb.Public.2006&spm=Jlcpcb.Instantquote&_t=1791669462672&up_spm=Jlcpcb.Loginpage.1003) | Main Board (Use gerbers provided) | 1 | $4.00 | $4.00 | [JLCPCB](https://cart.jlcpcb.com/quote?spm=jlcpcb.Public.2006&spm=Jlcpcb.Instantquote&_t=1791669462672&up_spm=Jlcpcb.Loginpage.1003) |
| [3.7V Battery](https://www.aliexpress.com/item/1005007316920489.html?spm=a2g0o.productlist.main.11.6908592fuu3elF&algo_pvid=89b3b27c-89f6-4d5d-aeda-8c8ed1f5e18e&algo_exp_id=89b3b27c-89f6-4d5d-aeda-8c8ed1f5e18e-10&pdp_ext_f=%7B%22order%22%3A%22146%22%2C%22eval%22%3A%221%22%2C%22fromPage%22%3A%22search%22%7D&pdp_npi=6%40dis%21NOK%2198.31%2157.02%21%21%2165.93%2138.24%21%400b15831117916714985933022e0def%2112000040233642089%21sea%21NO%218463637778%21X%211%210%21n_tag%3A-29911%3Bd%3A22d43e35%3Bm03_new_user%3A-29895&curPageLogUid=6zRNhpjZg3ha&utparam-url=scene%3Asearch%7Cquery_from%3A%7Cx_object_id%3A1005007316920489%7C_p_origin_prod%3A) | Powering the device while not connected (**Can be self funded**) | 1 | $0.01 | $0.01 | [AliExpress](https://www.aliexpress.com/item/1005007316920489.html?spm=a2g0o.productlist.main.11.6908592fuu3elF&algo_pvid=89b3b27c-89f6-4d5d-aeda-8c8ed1f5e18e&algo_exp_id=89b3b27c-89f6-4d5d-aeda-8c8ed1f5e18e-10&pdp_ext_f=%7B%22order%22%3A%22146%22%2C%22eval%22%3A%221%22%2C%22fromPage%22%3A%22search%22%7D&pdp_npi=6%40dis%21NOK%2198.31%2157.02%21%21%2165.93%2138.24%21%400b15831117916714985933022e0def%2112000040233642089%21sea%21NO%218463637778%21X%211%210%21n_tag%3A-29911%3Bd%3A22d43e35%3Bm03_new_user%3A-29895&curPageLogUid=6zRNhpjZg3ha&utparam-url=scene%3Asearch%7Cquery_from%3A%7Cx_object_id%3A1005007316920489%7C_p_origin_prod%3A) |
| [Passive Components](https://www.aliexpress.com/item/1005002489867848.html?spm=a2g0o.productlist.main.9.370926a8ctSQGM&algo_pvid=7c869774-9e38-45cb-95e6-598a8294238c&algo_exp_id=7c869774-9e38-45cb-95e6-598a8294238c-8&pdp_ext_f=%7B%22order%22%3A%225679%22%2C%22eval%22%3A%221%22%2C%22fromPage%22%3A%22search%22%7D&pdp_npi=6%40dis%21NOK%215.69%215.69%21%21%210.57%210.57%21%402103842a17917251852512610e0d09%2112000020846080146%21sea%21NO%218463637778%21X%211%210%21n_tag%3A-29911%3Bd%3A22d43e35%3Bm03_new_user%3A-29895&curPageLogUid=x1GYFxVaUCZz&utparam-url=scene%3Asearch%7Cquery_from%3A%7Cx_object_id%3A1005002489867848%7C_p_origin_prod%3A) | (Resistors, LED, Switch) (**Self provided**) | 1 | $0.01 | $0.01 | (Example part) [AliExpress](https://www.aliexpress.com/item/1005002489867848.html?spm=a2g0o.productlist.main.9.370926a8ctSQGM&algo_pvid=7c869774-9e38-45cb-95e6-598a8294238c&algo_exp_id=7c869774-9e38-45cb-95e6-598a8294238c-8&pdp_ext_f=%7B%22order%22%3A%225679%22%2C%22eval%22%3A%221%22%2C%22fromPage%22%3A%22search%22%7D&pdp_npi=6%40dis%21NOK%215.69%215.69%21%21%210.57%210.57%21%402103842a17917251852512610e0d09%2112000020846080146%21sea%21NO%218463637778%21X%211%210%21n_tag%3A-29911%3Bd%3A22d43e35%3Bm03_new_user%3A-29895&curPageLogUid=x1GYFxVaUCZz&utparam-url=scene%3Asearch%7Cquery_from%3A%7Cx_object_id%3A1005002489867848%7C_p_origin_prod%3A) |
| [Screws](https://www.aliexpress.com/item/33043885403.html?spm=a2g0o.productlist.main.1.704136c54twNW7&algo_pvid=0f932d31-3b09-495f-9419-6458a1488934&algo_exp_id=0f932d31-3b09-495f-9419-6458a1488934-0&pdp_ext_f=%7B%22order%22%3A%2214327%22%2C%22spu_best_type%22%3A%22price%22%2C%22eval%22%3A%221%22%2C%22fromPage%22%3A%22search%22%7D&pdp_npi=6%40dis%21NOK%2116.97%2116.97%21%21%211.70%211.70%21%400b1585c117917410156522141e0dd5%2112000020018123413%21sea%21NO%218463637778%21X%211%210%21n_tag%3A-29911%3Bd%3A22d43e35%3Bm03_new_user%3A-29895&curPageLogUid=5T3k97JHejJu&utparam-url=scene%3Asearch%7Cquery_from%3A%7Cx_object_id%3A33043885403%7C_p_origin_prod%3A) | For assembly of the enclosure, 4x self-tapping M3x8  and M2.5x8 (**Self provided**) | 1 | $0.01 | $0.01 | (Example part) [AliExpress](https://www.aliexpress.com/item/33043885403.html?spm=a2g0o.productlist.main.1.704136c54twNW7&algo_pvid=0f932d31-3b09-495f-9419-6458a1488934&algo_exp_id=0f932d31-3b09-495f-9419-6458a1488934-0&pdp_ext_f=%7B%22order%22%3A%2214327%22%2C%22spu_best_type%22%3A%22price%22%2C%22eval%22%3A%221%22%2C%22fromPage%22%3A%22search%22%7D&pdp_npi=6%40dis%21NOK%2116.97%2116.97%21%21%211.70%211.70%21%400b1585c117917410156522141e0dd5%2112000020018123413%21sea%21NO%218463637778%21X%211%210%21n_tag%3A-29911%3Bd%3A22d43e35%3Bm03_new_user%3A-29895&curPageLogUid=5T3k97JHejJu&utparam-url=scene%3Asearch%7Cquery_from%3A%7Cx_object_id%3A33043885403%7C_p_origin_prod%3A) |
| **Parts subtotal** | — | — | — | **$24.01** | — |
| **Tax & shipping** | — | — | — | **$6.97** | — |
| **Total** | — | — | — | **$30.98** | — |

**Please Note:** 
- For full transparency only the raw prices for each component have been listed, in reality significant discounts do apply for almost all of the components (Discounts will be shown in screenshots)
- Shipping has been disregarded for all components except for the PCB because of the free shipping offer applied to all choice orders above 10$
- The "first order discount" has been included in the price of the PCB, being taken away from the shipping cost (ie. 4$ PCB + 8.16$ shipping --> 4$ PCB + 4.16$ shipping), this has been done so that the project can still be submitted in the 30$ tier
- The keycaps for the switches will be 3D-printed so no listing has been provided
- I am willing to fund/provide all other components which do not fit into the 30$ range myself (Passive components that I already have, battery, and any other additional shipping and tax that may apply)


Cart Screenshots:


AliExpress Order (Excluding Self-funded parts) (1kr ≈ $0.1):

<img width="1181" height="911" alt="Screenshot 2026-10-11 154713" src="https://github.com/user-attachments/assets/2b54fa7d-e97a-40fa-8620-361473590f3d" />

Final payment window including tax:

<img width="379" height="316" alt="Screenshot 2026-10-11 154802" src="https://github.com/user-attachments/assets/17c6eec0-5185-4031-b3b1-6c8b5d8d6142" />

(129.24kr = **$13.52**)
<br>
<br>
<br>
Self-Funded part order (If the previous discounts are considered valid, then the extra money could still be used for these parts): 

<img width="1176" height="388" alt="Screenshot 2026-10-11 162357" src="https://github.com/user-attachments/assets/48063c6a-8552-4907-ad2a-aba513b133d9" />

Final payment window including tax:

<img width="373" height="353" alt="Screenshot 2026-10-11 162412" src="https://github.com/user-attachments/assets/0fed561d-aca2-473e-a989-ae6a1775ccbe" />

(24.53kr = **$2.57**)
<br>
<br>
<br>
JLCPCB Order:

<img width="1401" height="524" alt="Screenshot 2026-10-11 155750" src="https://github.com/user-attachments/assets/acf029ae-bf4f-4a1d-89cb-62d29947fcf1" />
<br>
<br>
--> Grand Total (with all discounts and self-funded parts): <b>$24.25<b>
