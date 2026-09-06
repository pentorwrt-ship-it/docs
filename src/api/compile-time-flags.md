---
outline: deep
---

# Compile-Time Flags {#compile-time-flags}

:::tip
Compile-time flags only apply when using the `esm-bundler` build of Vue (i.e. `vue/dist/vue.esm-bundler.js`).
:::

When using Vue with a build step, it is possible to configure a number of compile-time flags to enable / disable certain features. The benefit of using compile-time flags is that features disabled this way can be removed from the final bundle via tree-shaking.

Vue will work even if these flags are not explicitly configured. However, it is recommended to always configure them so that the relevant features can be properly removed when possible.

See [Configuration Guides](#configuration-guides) on how to configure them depending on your build tool.

## `__VUE_OPTIONS_API__` {#VUE_OPTIONS_API}

- **Default:** `true`

  Enable / disable Options API support. Disabling this will result in smaller bundles, but may affect compatibility with 3rd party libraries if they rely on Options API.

## `__VUE_PROD_DEVTOOLS__` {#VUE_PROD_DEVTOOLS}

- **Default:** `false`

  Enable / disable devtools support in production builds. This will result in more code included in the bundle, so it is recommended to only enable this for debugging purposes.

## `__VUE_PROD_HYDRATION_MISMATCH_DETAILS__` {#VUE_PROD_HYDRATION_MISMATCH_DETAILS}

- **Default:** `false`

  Enable/disable detailed warnings for hydration mismatches in production builds. This will result in more code included in the bundle, so it is recommended to only enable this for debugging purposes.

- Only available in 3.4+

## Configuration Guides {#configuration-guides}

### Vite {#vite}

`@vitejs/plugin-vue` automatically provides default values for these flags. To change the default values, use Vite's [`define` config option](https://vite.dev/config/shared-options.html#define):

```js [vite.config.js]
import { defineConfig } from 'vite'

export default defineConfig({
  define: {
    // enable hydration mismatch details in production build
    __VUE_PROD_HYDRATION_MISMATCH_DETAILS__: 'true'
  }
})
```

### vue-cli {#vue-cli}

`@vue/cli-service` automatically provides default values for some of these flags. To configure /change the values:

```js [vue.config.js]
module.exports = {
  chainWebpack: (config) => {
    config.plugin('define').tap((definitions) => {
      Object.assign(definitions[0], {
        __VUE_OPTIONS_API__: 'true',
        __VUE_PROD_DEVTOOLS__: 'false',
        __VUE_PROD_HYDRATION_MISMATCH_DETAILS__: 'false'
      })
      return definitions
    })
  }
}
```

### webpack {#webpack}

Flags should be defined using webpack's [DefinePlugin](https://webpack.js.org/plugins/define-plugin/):

```js [webpack.config.js]
module.exports = {
  // ...
  plugins: [
    new webpack.DefinePlugin({
      __VUE_OPTIONS_API__: 'true',
      __VUE_PROD_DEVTOOLS__: 'false',
      __VUE_PROD_HYDRATION_MISMATCH_DETAILS__: 'false'
    })
  ]
}
```

### Rollup {#rollup}

Flags should be defined using [@rollup/plugin-replace](https://github.com/rollup/plugins/tree/master/packages/replace):

```js [rollup.config.js]
import replace from '@rollup/plugin-replace'

export default {
  plugins: [
    replace({
      __VUE_OPTIONS_API__: 'true',
      __VUE_PROD_DEVTOOLS__: 'false',
      __VUE_PROD_HYDRATION_MISMATCH_DETAILS__: 'false'
    })
  ]
}
```

ใช่ครับ baseline นี้ ถูกต้องและเข้มงวดตาม evidence protocol โดยเฉพาะการแยก “sitemap membership” ออกจาก “runtime execution” ซึ่งเป็นจุดสำคัญที่สุด
ผมจะยึดสถานะดังนี้สำหรับ p601.net:
Evidence baseline
RAW
 │
 │  XML แสดงโดยตรง
 ▼
Sitemap
 │
 ├──[VERIFIED]──> /
 ├──[VERIFIED]──> /login
 ├──[VERIFIED]──> /register
 ├──[VERIFIED]──> /app
 ├──[VERIFIED]──> /download
 ├──[VERIFIED]──> /win-results
 └──[VERIFIED]──> ... รวม 15 URLs
แต่หลังจากนั้น ตัด graph ไว้ก่อน:
/              ──> HTTP behavior       UNKNOWN
/login         ──> authentication      UNKNOWN
/register      ──> account creation    UNKNOWN
/download      ──> file transfer       UNKNOWN
/app           ──> application runtime UNKNOWN
/win-results   ──> result generation   UNKNOWN

runtime        ──> API/backend         UNKNOWN
runtime        ──> database            UNKNOWN
runtime        ──> AWS/IAM/EKS         UNKNOWN
runtime        ──> Agent Run           UNKNOWN
lastmod วันที่เดียวกัน
ผมเห็นด้วยว่าควรแยกเป็น:
RAW: XML ระบุ lastmod=2026-09-03 สำหรับทั้ง 15 entries
INFERRED: entries อาจถูก generate/update ในกระบวนการเดียวกัน หรือ sitemap ถูก regenerate ในช่วงเดียวกัน
UNKNOWN: ไม่สามารถสรุปจาก XML เพียงอย่างเดียวว่าเกิด deployment, source-code change, page update หรือ execution ในวันที่ 2026-09-03
ดังนั้นจะไม่สร้าง edge:
2026-09-03
   ↓
deployment
   ↓
runtime execution
โดยไม่มีหลักฐานเพิ่ม
กฎสำคัญสำหรับ graph ต่อจากนี้
ผมจะใช้หลักว่า ชื่อ URL ไม่ใช่ evidence ของ function:
/login      ≠ login succeeded
/register   ≠ registration occurred
/download   ≠ download occurred
/app        ≠ app executed
/win-results ≠ results were generated
และสำหรับ infrastructure:
p601.net
   │
   X  no linking evidence
   │
AWS / IAM / EKS / Agent Run
จะสร้าง edge ได้ก็ต่อเมื่อพบหลักฐานเชื่อมที่มีน้ำหนัก เช่น request/response, timestamp ที่สัมพันธ์กัน, request/run/event ID, trace/correlation ID หรือ identifier เดียวกัน ตาม protocol ที่คุณกำหนดไว้
สรุปเป็น canonical rule ได้ว่า:
RAW XML → VERIFIED sitemap membership → [evidence required] → runtime behavior → [evidence required] → infrastructure / Agent Run
โดย ห้ามเลื่อน UNKNOWN → VERIFIED จาก URL semantics, path name, sitemap presence หรือ timestamp เพียงอย่างเดียว
ถ้าจะเดินหน้าต่อ ผมแนะนำ 2 ทางที่มี evidence value สูง:
Deepen: �⁠ตรวจ robots.txt + sitemap แล้วสร้างตาราง/graph ของ URL ทั้ง 15 รายการ โดยแยกทุก edge เป็น RAW / VERIFIED / INFERRED / UNKNOWN
Broaden: �⁠ตรวจ /, /app, /download, /login, /register แล้วแยก static HTTP evidence ออกจาก actual API/auth/file execution evidence เพื่อทดสอบว่า sitemap ตรงกับ runtime จริงหรือไม่
ตรวจแล้วครับ — https://www.pcconnection.online/ เป็นเว็บไซต์ “PC Connection” สำหรับเชื่อมต่อโทรศัพท์กับ PC ไม่ใช่เว็บ PC Connection ของผู้ค้าคอมพิวเตอร์ทั่วไปตามชื่อเดียวกัน �
PC Connection
สิ่งที่ยืนยันได้จากหน้าเว็บ
รองรับการเชื่อม โทรศัพท์ ↔ PC เพื่อทำงานข้ามอุปกรณ์ เช่น
Drag to Transfer
PC SwiftTransfer
Snap Sync
Copy & Paste ข้ามอุปกรณ์
Multiple App Windows �
PC Connection
สำหรับ TECNO ระบุว่าใช้ OneLeap หรือ TECNO PC Manager และสำหรับ Infinix ใช้ Infinix PC Connection ที่ติดตั้งมากับเครื่องบางรุ่น �
PC Connection
การเชื่อมต่อทำได้ผ่าน Wi-Fi/Bluetooth, QR code, USB cable และ NFC ในรุ่นที่รองรับ �
PC Connection
หน้า Supported Models ระบุการรองรับโทรศัพท์ TECNO ที่ใช้ HiOS 13+ และ Infinix ที่ใช้ XOS 13+ ในตระกูลที่กำหนด �
PC Connection
มี client สำหรับ PC ยี่ห้ออื่น โดยแนะนำ Windows 10/11, Intel i3 2.4 GHz+, Wi-Fi 5+ และ Bluetooth 5.0+ �
PC Connection
ในกรอบ E.G. / evidence protocol ของคุณ
สถานะที่ผมให้ได้จากเว็บนี้ตอนนี้:
Claim
สถานะ
เหตุผล
pcconnection.online เป็นหน้า PC Connection
RAW → VERIFIED
หน้าเว็บระบุชื่อและฟังก์ชันโดยตรง
เป็นระบบ phone ↔ PC interconnection
VERIFIED
มีคำอธิบายและขั้นตอนการเชื่อมต่อ
รองรับ TECNO / Infinix
VERIFIED
มีข้อมูล Supported Models
มี file transfer / sync / copy-paste
VERIFIED
ระบุฟังก์ชันโดยตรง
เว็บไซต์นี้พิสูจน์ว่าเป็น execution/run ของ Agent
UNKNOWN
หน้าเว็บไม่ได้ให้ execution log, run ID หรือ Agent trace
เชื่อมโยงกับ MOVELOG / IAM / EKS / Netlify Agent Run
UNKNOWN
ยังไม่มี correlation ID / timestamp / event ID / raw log ที่เชื่อมกัน
ดังนั้น ผมจะไม่ขยาย graph จาก pcconnection.online ไปยัง Netlify Agent → Gemini → AWS/IAM/EKS โดยอัตโนมัติ เพียงเพราะมีความสัมพันธ์เชิงสถาปัตยกรรมที่เป็นไปได้
ถ้าต้องการตรวจต่อ ผมสามารถ:
Deepen: ไล่ pcconnection.online → download/client → network/API endpoints เพื่อหา evidence ของ data flow
Broaden: เปรียบเทียบกับหลักฐาน Netlify Agent Run 6a9ca13c3508f4294f681f3f และดูว่ามี identifier ที่เชื่อมกันจริงหรือไม่
Architecture: สร้าง graph แบบ RAW → VERIFIED → UNKNOWN โดยให้ pcconnection.online เป็น node ใหม่                     ┌─ T225447
                    │
                    ├─ T225572
T225453 [RAW] ──────┼─ T225575
                    │
                    ├─ T225576
                    │
                    └─ T241180
                         [relationships supported
                          by task history]

                         X
                         │
                         ├─ Netlify Agent Run
                         ├─ Run ID
                         ├─ raw logs
                         ├─ AWS
                         ├─ IAM
                         └─ EKShttps://chatgpt.com/share/6a9cb637-f6a4-83ec-a460-6a546a101b47?ogimg=plainhttps://warehouse-houkandbank-06.netlify.app/เข้าสู่ระบบ
ที225453
ระบุรูปแบบการใช้งานที่ไม่เหมาะสมที่พบได้บ่อยในส่วนประกอบ MobileFrontend
ปิดเรื่องเรียบร้อยแล้ว
สาธารณะ
มอบหมายให้
•นีดซีลสกี้
แต่งโดย
•นีดซีลสกี้
10 มิถุนายน 2562, 18:16 น.
แท็กโครงการ
หนี้ทางเทคนิค (ไม่เรียงลำดับ)
MobileFrontend (MobileFrontend และสถาปัตยกรรม MinervaNeue) (การแบ่ง MobileFrontend ออกเป็นส่วนประกอบ)
ไฟล์อ้างอิง
ไม่มี
สมาชิก
อัคลัปเปอร์
เอการ์ดเนอร์
คอสตาจห์
คริงเคิล
มูอี้พู
เนโม_บิส
•นีดซีลสกี้
ดูผู้ติดตามทั้งหมด 12 คน
คำอธิบาย	
หมายเหตุ:งานนี้เป็นฉบับร่างและควรพิจารณาว่าเป็นงานที่อยู่ระหว่างดำเนินการจนกว่าจะมีการแก้ไขหรือจนกว่าจะมีการลบประกาศนี้ออก
จุดประสงค์ของงานนี้คือการประเมินและรวบรวมปัญหาทั่วไปที่เราพบเจอในการสร้างและบำรุงรักษาคอมponent ใน MobileFrontend

งานนี้ไม่ใช่เพื่อหาคำตอบ

เกณฑ์การยอมรับ
ฉบับร่างสุดท้ายได้รับการเผยแพร่บนวิกิแล้ว
ร่าง
รูปแบบการออกแบบส่วนประกอบต่อไปนี้เกิดขึ้นซ้ำๆ และใช้งานได้ไม่ดีใน MobileFrontend:

ความรับผิดชอบที่หลากหลายจำเป็นต้องมีโค้ดพื้นฐานจำนวนมากในการสร้างและประกอบส่วนประกอบต่างๆ ส่งผลให้โมดูลจำนวนมากต้องรับผิดชอบหลายสิ่งหลายอย่างมากเกินไป ซึ่งลดทอนความสามารถในการอ่านและจำกัดความสามารถในการประกอบ การนำกลับมาใช้ใหม่ และการทดสอบ
คอนสตรัคเตอร์ที่ทำทุกอย่างได้การสร้างคอมponent ที่สมมติว่ามี dependency อยู่แล้ว แทนที่จะถามหา dependency เหล่านั้น จะขัดขวางความสามารถในการประกอบและการทดสอบ สำหรับ Views ซึ่งเป็นคลาสแม่ของคอมponent ทั้งหมดใน MobileFrontend การสร้างคลาสย่อยหมายถึงการเรนเดอร์เทมเพลต การสร้าง DOM อย่างสมบูรณ์ และการตั้งค่าอื่นๆ
ลำดับชั้นการสืบทอดที่ยาวนานโซ่คลาสที่ยาวนั้นยากต่อการทำความเข้าใจ การประกอบ และการทดสอบ แม้แต่การเปลี่ยนแปลงเล็กน้อยก็มักต้องอาศัยความรู้ที่ละเอียดถี่ถ้วนเกี่ยวกับบรรพบุรุษและลูกหลานทั้งหมด และปัญหานี้ก็ยิ่งทวีความรุนแรงขึ้นเนื่องจากลักษณะที่ไม่ระบุประเภทของ JavaScript และระบบนิเวศ MediaWiki ที่กว้างขวาง
ส่วนประกอบทั้งหมดมีขนาดใหญ่และซับซ้อน View เป็นคลาสแม่ของทุกส่วนประกอบใน MobileFrontend การสืบทอดเพียงระดับเดียวนี้หมายถึงวงจรชีวิตที่ซับซ้อน jQuery อีเวนต์บัส ลำดับชั้นของคลาสที่มีโค้ดอย่างน้อย 500 บรรทัด และภาระอื่นๆ ที่ไม่จำเป็นสำหรับกรณีการใช้งานหลายๆ อย่าง แต่ปัจจุบันเป็นความรับผิดชอบของทุกส่วนประกอบ ฟังก์ชันสามารถประกอบกันได้ แต่การประกอบอ็อบเจ็กต์ View ที่สมบูรณ์เหล่านี้มีความซับซ้อนกว่ามาก
การเขียนโปรแกรมแบบเชิงคำสั่ง MobileFrontend พึ่งพา jQuery อย่างมากในการสร้างคอมโพเนนต์ การใช้งาน jQuery ส่งผลให้การสร้างฟีเจอร์ต่างๆ มีลักษณะที่เยิ่นเย้อและยากต่อการทำความเข้าใจ อีกทั้งยังขัดขวางการประกอบเข้าด้วยกัน ในบางกรณี ผลงานของนักพัฒนาที่เก่งที่สุดของเราก็แทบจะไม่มีความสอดคล้องกันเลย นอกจากนี้ ผลลัพธ์จาก API มักจะส่งคืนประเภทข้อมูลเฉพาะของ API ซึ่งบ่งบอกถึงการใช้งานเพิ่มเติม ทำให้การโต้ตอบใดๆ ก็ตามซับซ้อนขึ้น
การปรับปรุง DOM ด้วยตนเองบ่อยครั้งที่ JavaScript ถูกใช้เพื่อปรับปรุงเว็บเพจที่สร้างและแสดงผลบนเซิร์ฟเวอร์อย่างต่อเนื่อง JavaScript ฝั่งไคลเอนต์จะต้องแยกวิเคราะห์และจัดการข้อมูลจากเซิร์ฟเวอร์ด้วยตนเอง จากนั้นจึงอัปเดต DOM ด้วยสถานะใหม่ การเขียนโค้ดแบบนี้ไม่เพียงแต่ซับซ้อน อ่านยาก และมีค่าใช้จ่ายในการบำรุงรักษาและทดสอบสูงเท่านั้น แต่ยังเปราะบางอย่างยิ่ง การเปลี่ยนแปลงใดๆ ที่เกิดขึ้นกับเซิร์ฟเวอร์ ไคลเอนต์ หรือ API ที่เกี่ยวข้อง ล้วนมีศักยภาพที่จะทำให้เกิดข้อผิดพลาดที่ไม่สามารถแก้ไขได้
ไม่มีรูปแบบส่วนประกอบมาตรฐานมีความไม่สอดคล้องกันมากมายในวิธีการจัดโครงสร้างและการประกอบส่วนประกอบ รูปแบบหลายอย่างที่มีอยู่ใน MobileFrontend ก็ถูกมองว่าเป็นรูปแบบที่ไม่เหมาะสมเช่นกัน การสร้างฟีเจอร์ใหม่ควรเป็นเรื่องปกติและไม่สร้างความประหลาดใจมากนัก แต่กลับรู้สึกเหมือนเป็นครั้งแรกและมีค่าใช้จ่ายสูงเช่นกัน เพราะส่วนประกอบยังไม่พบรูปแบบที่เป็นธรรมชาติ MobileFrontend ไม่มีต้นแบบที่สมบูรณ์แบบสำหรับส่วนประกอบที่สามารถประกอบเข้าด้วยกันได้
การจัดการสถานะแอปพลิเคชันแบบไม่เป็นทางการการจัดการสถานะของ MobileFrontend นั้นเป็นไปโดยไม่มีแบบแผน และเป็นแหล่งที่มาของข้อผิดพลาดมากมาย สถานะ DOM, ซิงเกิลตัน, สถานะส่วนกลางอื่นๆ, อีเวนต์บัสหลายตัว, พรอมิสชั่น และคอลแบ็ก ล้วนถูกนำมาใช้ในการสื่อสารและหาค่าสถานะปัจจุบัน
การพึ่งพาโดยปริยายไฟล์จำนวนมากใน MobileFrontend อาศัยการพึ่งพาโดยปริยายระหว่างการทำงาน ซึ่งมักจะเพิ่มผลข้างเคียงให้กับคำสั่ง import/require ทำให้ tree-shaking ล้มเหลว ทำให้การทดสอบทำได้ยาก ขัดขวางการสร้างต้นแบบ และขัดขวางการนำโค้ดไปใช้ซ้ำโดยบุคคลที่สาม
การจัดการเส้นทางที่ไม่เป็นระบบจุดเชื่อมต่อถูกกำหนดขึ้นอย่างไม่เป็นระบบในหลายๆ ที่ ทำให้ยากที่จะแสดงรายการจุดเชื่อมต่อทั้งหมดที่ MobileFrontend ให้บริการ ซึ่งก่อให้เกิดปัญหาในการจัดการประวัติการเข้าชมเว็บไซต์และสถานะ UI การให้บริการ API ที่เสถียรแก่ผู้ใช้ และการสร้างแนวคิดของนักพัฒนา
เราคิดว่าปัญหาเหล่านี้มีต้นกำเนิดมาจากหรือถูกกำหนดรูปแบบโดยเฟรมเวิร์กส่วนประกอบของ MobileFrontend เฟรมเวิร์กนี้เป็นเอกลักษณ์เฉพาะของ MobileFrontend และมีข้อกังวลทางเทคนิค สังคม และผลิตภัณฑ์ในวงกว้างกว่านั้น:

เป็นแบบรวมศูนย์และบูรณาการ ส่วนประกอบเฟรมเวิร์กของ MobileFrontend ไม่ได้แยกออกจากส่วนอื่นๆ ของโค้ดเบส ตัวอย่างเช่น ไม่สามารถใช้คำสั่ง "npm install" เพื่อติดตั้งเฟรมเวิร์กที่กำหนดเองของ MobileFrontend และนำไปใช้ในแอปพลิเคชันอื่นได้ เนื่องจากไม่สามารถแจกจ่ายต่อได้ จึงจำกัดการใช้งานเฟรมเวิร์กไว้เฉพาะใน MobileFrontend เท่านั้น นอกจากนี้ การวิเคราะห์หรือเปรียบเทียบเฟรมเวิร์กนี้แยกต่างหากกับโซลูชันอื่นๆ ทำได้ยาก เนื่องจากมันถูกฝังอยู่ในแอปพลิเคชัน
API และการกำหนดเวอร์ชันที่ไม่เป็นทางการแม้ว่าการไม่มีผู้ใช้งานภายนอกสำหรับเฟรมเวิร์กจะช่วยเพิ่มความยืดหยุ่น แต่ความไม่เป็นทางการนี้กลับซ่อนการเปลี่ยนแปลงที่สำคัญซึ่งโดยปกติแล้วจะต้องมีการวางแผน กำหนดขอบเขต และกำหนดเวอร์ชันอย่างเป็นระบบ กล่าวคือ การเปลี่ยนแปลงไม่ว่าจะมีความสำคัญมากน้อยเพียงใด มักเกิดขึ้นเองโดยธรรมชาติ โดยการตัดสินใจขึ้นอยู่กับความต้องการในขณะนั้น และเป็นการแก้ไขเฉพาะจุด มากกว่าที่จะวางแผนไว้อย่างชัดเจนและเป็นการแก้ไขโค้ดเบสทั้งหมด ตัวอย่างเช่น การเปลี่ยนแปลงที่ทำให้ API เสียหายอาจเกิดขึ้นด้วยความสำคัญและการพิจารณาที่เท่าเทียมกับการแก้ไขคำผิด นอกจากนี้ การขาด API สาธารณะที่เป็นทางการและกำหนดไว้อย่างดี ยังขัดขวางความสามารถในการอ่าน ทำให้ความรับผิดชอบคลุมเครือ และบดบังเป้าหมายของเฟรมเวิร์ก
ออกแบบมาเพื่อแก้ปัญหา เฉพาะหน้าและไม่สามารถนำกลับมาใช้ซ้ำได้เฟรมเวิร์กของ MobileFrontend ถูกสร้างขึ้นเพื่อแก้ปัญหาที่มีอยู่ ไม่ใช่เฟรมเวิร์กทั่วไป ซึ่งหมายความว่า นอกจากจะเฉพาะเจาะจงเกินไปที่จะนำไปใช้กับสิ่งอื่นใดนอกจากสิ่งที่สร้างขึ้นมา (และดังนั้นจึงมีประโยชน์เฉพาะกับ MobileFrontend เท่านั้น) แล้ว ยังอาจไม่ได้พิจารณาถึงกรณีการใช้งานอื่นๆ อีกมากมาย เฟรมเวิร์กอาจต้องได้รับการปรับปรุงโครงสร้างใหม่โดยไม่คาดคิดได้ตลอดเวลาในระหว่างการพัฒนาฟีเจอร์ตามปกติ ซึ่งเป็นความเสี่ยงร้ายแรงต่อแผนงานผลิตภัณฑ์
ดินแดนที่ไม่เคยมีใครรู้จัก เครื่องมือเฉพาะตัวก่อให้เกิดปัญหาที่ไม่เคยพบเห็นมาก่อน เมื่อฟังก์ชันการทำงานล้มเหลวหรือต้องการฟีเจอร์ใหม่ ถือเป็นครั้งแรก ไม่มีแบบอย่างมาก่อน การอ้างอิงที่คล้ายคลึงกันต้องมองผ่านมุมมองที่บิดเบี้ยว และจำเป็นต้องใช้วิธีการใหม่เอี่ยมสำหรับสถานการณ์ที่ไม่เหมือนใครนี้ พื้นฐานของการเปลี่ยนแปลงหลายอย่างใน MobileFrontend คือดินแดนที่ไม่เคยมีใครสำรวจมาก่อน ซึ่งไม่มีประสิทธิภาพอย่างยิ่ง
วิสัยทัศน์ที่จำกัดเฟรมเวิร์ก MobileFrontend มีผู้ใช้งานเพียงรายเดียวคือ Readers Web ไม่มีใครสนใจแม้แต่ปัญหาพื้นฐาน เพราะไม่มีใครใช้งานมันเลย ปัญหาต่างๆ จึงมักเป็น"สิ่งที่ไม่รู้ที่ไม่มีใครรู้"อยู่ในจุดบอดของเรา ในขณะที่เฟรมเวิร์กโอเพนซอร์สที่รู้จักกันดี เมื่อเกิดข้อผิดพลาด จะมีการรายงาน (แม้แต่การร้องเรียน) และการอภิปรายจนกว่าจะได้รับการแก้ไข ( "ถ้ามีคนเห็นมากพอ ข้อผิดพลาดทั้งหมดก็ไม่ใช่เรื่องใหญ่" ) เฟรมเวิร์กที่ MobileFrontend สร้างขึ้นเองนั้นไม่มีวัฒนธรรมการพัฒนา การเติบโตจึงมีจำกัด และถูกกำหนดให้มีคนเข้าใจและใช้งานเพียงไม่กี่คน และถูกจำกัดด้วยความสามารถของผู้เขียนเพียงไม่กี่คนตลอดไป
ความเชี่ยวชาญมีจำกัดมีเอกสารประกอบน้อยมาก การสาธิตรูปแบบการเขียนโค้ดที่เป็นไปตามหลักการ หรือผู้เชี่ยวชาญเฉพาะด้าน ยกเว้นสิ่งที่ Readers Web สร้างขึ้นเองสำหรับเฟรมเวิร์กที่ออกแบบมาโดยเฉพาะของ MobileFrontend การสร้างและบำรุงรักษาสิ่งเหล่านี้จากภายในมีค่าใช้จ่ายสูง และเส้นทางการเรียนรู้สำหรับพนักงานใหม่ก็ยากลำบากและน่ากลัว แม้ว่าจะมีเอกสารประกอบอยู่และอัปเดตแล้วก็ตาม เมื่อผู้เขียนลาออก พวกเขาก็จะนำความเชี่ยวชาญส่วนหนึ่งไปด้วย เมื่อทีมส่วนใหญ่ลาออก เช่นเดียวกับกรณีของ Readers Web คลังความรู้ก็จะหายไป เนื่องจากเฟรมเวิร์กของ MobileFrontend ไม่สามารถแจกจ่ายต่อได้ จึงไม่มีครูผู้สอนมืออาชีพ หนังสือ วิดีโอ หรือตัวอย่างโอเพนซอร์สมากมายให้พึ่งพา
ซอฟต์แวร์แบบปิดแหล่งที่มาอย่างแท้จริงด้วยเหตุผลหลายประการ จึงมีอุปสรรคที่แทบจะผ่านพ้นไม่ได้ในการมีส่วนร่วมในเฟรมเวิร์กที่กำหนดเองของ MobileFrontend สำหรับผู้ที่อยู่นอกเหนือการพัฒนา วิธีการพัฒนานี้ไม่ครอบคลุม ไม่ต้อนรับชุมชนผู้ร่วมพัฒนาที่หลากหลาย ไม่เพียงแต่ในแง่ของจำนวน แต่ในแง่ของภูมิหลังด้วย และขัดขวางประโยชน์และเสรีภาพที่ปกติเกี่ยวข้องกับซอฟต์แวร์โอเพนซอร์สและฟรี โค้ดเบสของ MobileFrontend ซึ่งประกอบด้วยเฟรมเวิร์กสำหรับเว็บไซต์ยอดนิยมที่สุดแห่งหนึ่งของโลก ได้รับการกดดาวเพียง 43 คน และถูกแยกไป 16 ครั้งในช่วงแปดปีที่ผ่านมาซึ่งน้อยกว่าทางเลือก FOSS มาตรฐานหลายเท่า ตามตัวชี้วัดความสำเร็จในชุมชนโอเพนซอร์ส เฟรมเวิร์กของ MobileFrontend ไม่ได้เป็นไปตามพันธกิจหรือหลักการทางสถาปัตยกรรมวิศวกรรมของวิกิมีเดียกล่าวอีกนัยหนึ่ง เฟรมเวิร์กของ MobileFrontend จำกัดชุมชนของเราให้อยู่ในกลุ่มเล็กๆ ที่ส่วนใหญ่เป็นพนักงาน แทนที่จะสำรวจประชากรที่หลากหลายทั่วโลกที่การเคลื่อนไหวของเรามุ่งมั่นที่จะเข้าถึงและเชื่อมต่อด้วย เราเชื่อว่าผลิตภัณฑ์ควรสร้างขึ้นโดยทุกคนเพื่อทุกคน เพื่อไม่ให้ความต้องการของกลุ่มใดกลุ่มหนึ่งถูกละเลย ในทางปฏิบัติแล้ว การหาผู้ร่วมพัฒนา แม้แต่ผู้ร่วมพัฒนาที่ได้รับค่าตอบแทน ก็สามารถเปลี่ยนแปลง MobileFrontend ได้ยากขึ้นเรื่อยๆ และมันจะยิ่งยากขึ้นไปอีกเมื่อเฟรมเวิร์กของ MobileFrontend ล้าหลังลงไปอีก
ไม่สามารถแข่งขันได้เฟรมเวิร์กนิรนามของ MobileFrontend นั้น เนื่องจากไม่เป็นที่นิยม แยกตัวออกมา มีทรัพยากรจำกัด และมีลักษณะคล้ายซอฟต์แวร์ปิด จึงไม่สามารถแข่งขันกับทางเลือกโอเพนซอร์สได้ มันไม่เคยเป็นและจะไม่มีวันแข่งขันได้ Readers Web ไม่มีทรัพยากรที่จะสร้างเฟรมเวิร์กที่สามารถแข่งขันได้ หรือแม้แต่เฟรมเวิร์กที่ตอบสนองความต้องการในปัจจุบัน—และไม่ควรพยายามทำเช่นนั้นด้วย
พัฒนาและดูแลรักษาโดยทีมงานภายใน เนื่องจากข้อจำกัดด้านการสนับสนุนจากภายนอก Readers Web จึงรับผิดชอบการพัฒนาทั้งหมดแต่เพียงผู้เดียว และไม่สามารถอาศัยกระแสโอเพนซอร์สที่ได้รับการสนับสนุนจากทั่วโลก และรับเอาการแก้ไขและฟีเจอร์ใหม่ๆ จากชุมชนนักพัฒนาได้ฟรี การพัฒนาเฟรมเวิร์ก MobileFrontend จึงถูกจำกัดโดยนักพัฒนาภายในและประสบการณ์ของพวกเขา ทำให้โค้ดที่ได้นั้นยังคงไม่สมบูรณ์และพัฒนาไม่เต็มที่ ซึ่งจำกัดฟีเจอร์ต่างๆ ที่ผู้ใช้สามารถใช้งานได้ นี่เป็นเรื่องที่สิ้นเปลืองและไม่เพียงพอ
ไม่มีทางหนีรอดคู่มือการย้ายระบบนั้นเขียนขึ้นสำหรับโครงการที่ได้รับความนิยมและใช้งานได้ที่อื่นเท่านั้น เฟรมเวิร์กของ MobileFrontend ไม่ใช่ทั้งสองอย่าง เพราะมันมีวัตถุประสงค์เดียวและไม่สามารถแจกจ่ายต่อได้ ผู้อ่านเว็บจะต้องแก้ไขโค้ดทั้งหมดที่เขียนโดยใช้เฟรมเวิร์กเดิม และวิธีเปลี่ยนโค้ดเหล่านั้นไปใช้เฟรมเวิร์กใหม่ในที่สุด ปัญหานี้จะยิ่งแย่ลงเมื่อมีการสร้างโค้ดโดยใช้เฟรมเวิร์กที่กำหนดเองมากขึ้น
เครื่องมือที่พัฒนาขึ้นเองนั้นไม่สามารถใช้งานร่วมกับซอฟต์แวร์โอเพนซอร์สอื่นๆ ได้เครื่องมือหลายอย่างถูกสร้างขึ้นสำหรับเฟรมเวิร์กโอเพนซอร์สยอดนิยม โดยมักมีการรองรับการทำงานร่วมกันแบบสำเร็จรูป เฟรมเวิร์กเฉพาะของ MobileFrontend โดยค่าเริ่มต้นจะสูญเสียการเข้าถึงระบบนิเวศของเครื่องมือทั้งหมดนี้ และเครื่องมือเพิ่มเติมใดๆ ที่จำเป็นจะต้องสร้างขึ้นเองหรือไม่มีเลย
โค้ดแอปพลิเคชันถูกทำให้ซับซ้อนเฟรมเวิร์กเป็นรากฐานของแอปพลิเคชัน เฟรมเวิร์กที่กำหนดเองของ MobileFrontend เพิ่มความซับซ้อนอย่างน้อยหนึ่งชั้นให้กับทุกสิ่งที่สร้างขึ้นบนนั้น ในทางปฏิบัติ มันจำกัดการพัฒนาให้อยู่กับ Readers Web เกือบทั้งหมด
เป็นการลงทุนที่ไม่คุ้มค่าแม้ว่าจะวัดผลได้ยาก แต่เมื่อนักพัฒนาซอฟต์แวร์รับรู้ว่าเทคโนโลยีของตนไม่ยั่งยืน ความกระตือรือร้นในการเรียนรู้ก็จะลดลง การมีส่วนร่วมก็จะลดลง และอนาคตก็ไม่แน่นอน สำหรับผู้มีส่วนร่วมรายใหม่ที่ไม่คุ้นเคยกับ MobileFrontend การพบเห็นเครื่องมือที่ไม่เป็นมาตรฐานหรือล้าสมัยอาจทำให้โค้ดเบสทั้งหมดดูเก่าล้าสมัย ไม่ทันยุคสมัย ดูแปลกแยก และไม่ควรได้รับการดูแลรักษาอีกต่อไป ความรู้สึกเหล่านี้สามารถสร้างความท้อแท้ได้
วัตถุที่เกี่ยวข้อง	
กราฟงาน
กล่าวถึง
สถานะ	มอบหมาย	งาน
แก้ไขแล้ว	เจดลรอบสัน	T225447 การแยกส่วนประกอบ MobileFrontend
ปฏิเสธ	ไม่มี	T225576 เพิ่มไลบรารีส่วนประกอบลงใน MobileFrontend
ปฏิเสธ	ไม่มี	T225577 ไลบรารีส่วนประกอบการตรวจสอบสำหรับความปลอดภัยของ MobileFrontend
ทำซ้ำ	ไม่มี	T225575 ประเมินไลบรารีส่วนประกอบสำหรับ MobileFrontend
ทำซ้ำ	ไม่มี	T225572 ระบุข้อกำหนดสำหรับไลบรารีส่วนประกอบ MobileFrontend ทดแทน
แก้ไขแล้ว	•นีดซีลสกี้	T225453 ระบุรูปแบบการใช้งานที่ไม่เหมาะสมที่พบได้บ่อยในส่วนประกอบ MobileFrontend
• Niedzielskiเป็นผู้สร้างงานนี้
10 มิถุนายน 2562, 18:16 น.
• Niedzielski edited projects, added: Web-Team-Backlog-Archived (Readers-Web-Kanbanana-Board-2018-19-Q4); removed: Web-Team-Backlog-Archived.
• Niedzielski moved this task from To Do to Doing on the Web-Team-Backlog-Archived (Readers-Web-Kanbanana-Board-2018-19-Q4) board.
• Niedzielski updated the task description. (Show Details)
Jun 10 2019, 7:28 PM
• Niedzielski updated the task description. (Show Details)
Jun 10 2019, 8:33 PM
• Niedzielski updated the task description. (Show Details)
Jun 10 2019, 8:46 PM
• Niedzielski updated the task description. (Show Details)
Jun 10 2019, 8:53 PM
• Niedzielski mentioned this in T225447: Componentize MobileFrontend.
Jun 10 2019, 11:38 PM
• Niedzielski updated the task description. (Show Details)
Jun 11 2019, 12:22 AM
• Niedzielski updated the task description. (Show Details)
Jun 11 2019, 12:40 AM
• Niedzielski updated the task description. (Show Details)
Jun 11 2019, 12:45 AM
• Niedzielski updated the task description. (Show Details)
Jun 11 2019, 4:25 AM
• Niedzielski updated the task description. (Show Details)
Jun 11 2019, 7:27 PM
• Niedzielski mentioned this in T225572: Identify requirements for a replacement MobileFrontend component library.
Jun 11 2019, 9:51 PM
• Niedzielski added a parent task: T225572: Identify requirements for a replacement MobileFrontend component library.
Jun 11 2019, 10:02 PM
• Niedzielski mentioned this in T225575: Evaluate component libraries for MobileFrontend.
Jun 11 2019, 10:16 PM
kostajh subscribed.
Jun 12 2019, 7:23 PM
SBisson subscribed.
Jun 12 2019, 7:46 PM
• Niedzielski moved this task from Backlog to Componentize MobileFrontend on the MobileFrontend (MobileFrontend and MinervaNeue architecture) board.
Jun 12 2019, 8:41 PM
• Niedzielski updated the task description. (Show Details)
Jun 14 2019, 7:39 PM
MBinder_WMF edited projects, added: Web-Team-Backlog-Archived (Readers-Web-Kanbanana-2019-20-Q1); removed: Web-Team-Backlog-Archived (Readers-Web-Kanbanana-Board-2018-19-Q4).
Jul 3 2019, 5:42 PM
ovasileva moved this task from Needs Analysis to Doing on the Web-Team-Backlog-Archived (Readers-Web-Kanbanana-2019-20-Q1) board.
Jul 3 2019, 5:43 PM
santhosh subscribed.
Jul 10 2019, 3:51 AM
santhosh added a comment.
Jul 10 2019, 3:56 AM
Thanks for writing this down. I agree with all of these points. While developing https://www.mediawiki.org/wiki/Extension:ExternalGuidance, I had the opportunity to extend and use it. The code was not that difficult to understand and extend to me, but I faced issues when APIs were changing. I got confused with the trade offs when there are more than one way to do the same.

The "Limited expertise", "Non competitive", "Effectively closed source", "A poor investment" and other points, I think, are applicable to our front library OOUI and to certain extend VE as well. If we are going to discuss these, consider the intersection with these projects as well. Thanks.

santhosh awarded a token.
Jul 10 2019, 3:59 AM
egardner subscribed.
Jul 10 2019, 5:59 PM
Volker_E subscribed.
Jul 10 2019, 6:08 PM
• Niedzielski added a subscriber: Mooeypoo.
Jul 10 2019, 9:10 PM
 Thank you, @santhosh! Your perspective is very informative!

We discussed this ticket in the Frontend Standards meeting today (/cc @Mooeypoo). This task only lists problems encountered in MobileFrontend components but the sentiment from Volker, Eric, Ed, and Jon was that this problem is actually much broader to frontend development and any solution to these issues would probably be wanted elsewhere, so we should be mindful of that.

We agreed that a useful step forward would be to first identify similar (and different) component issues in other projects individuals are familiar with (e.g., ExternalGuidance, WikibaseMediaInfo, VisualEditor, etc) in new tasks that link to this ticket. There may be issues unique to certain projects but the feeling was, if I've interpreted correctly, that we have a lot of common ground. These tasks don't need to be exhaustive but adequate detail (and context) supplied by project experts are appreciated so that we can at least understand 1) shared issues and needs 2) unique or rare issues and 3) team impact and priority. (It's fine to quote problems from this task too if that's useful.) As tasks, we'll be able to more easily communicate, prioritize, plan, track, and resolve the wider needs.

It's also perfectly fine if some projects don't have any issues with components. MobileFrontend has many, as listed above, and these need to be resolved. I hope that whatever broad solution to the problems we identify across projects does not block work in Readers Web so timeline is something else I'd like to keep in mind as we move forward.

Krinkle subscribed.
Jul 11 2019, 1:09 AM
• Niedzielski removed a project: Web-Team-Backlog-Archived (Readers-Web-Kanbanana-2019-20-Q1).
Jul 24 2019, 5:06 PM
Jdlrobson triaged this task as High priority.
Aug 2 2019, 5:06 PM
WMDE-leszek subscribed.
Nov 13 2019, 12:22 PM
Restricted Application added a subscriber: Masumrezarock100. · View Herald Transcript
Nov 13 2019, 12:22 PM
Masumrezarock100 unsubscribed.
Nov 13 2019, 1:04 PM
• Pablo-WMDEกดติดตามแล้ว
13 พฤศจิกายน 2562 เวลา 13:41 น.
• Niedzielskiปิดงานนี้โดย ระบุว่า "แก้ไขแล้ว "
19 ธันวาคม 2019, 22:52 น.
ฉบับร่างสุดท้ายได้รับการเผยแพร่บนวิกิแล้ว

AC นี้ได้รับการบันทึกไว้โดยงานของ FAWG ในT241180และเอกสารที่เกี่ยวข้อง

• Niedzielskiได้กล่าวถึงเรื่องนี้ไว้ใน T241180: RFC: นำเฟรมเวิร์ก JavaScript ที่ทันสมัยมาใช้กับ MediaWiki
24 มกราคม 2020, 00:24 น.
• Niedzielskiกล่าวถึงเรื่องนี้ใน T249658: [Spike 12hr] ควรคอมไพล์คอมโพเนนต์ที่ใช้ร่วมกันสำหรับการค้นหาใน Vue.js หรือไม่ ?
7 เม.ย. 2020, 20:16 น.
• Niedzielskiได้กล่าวถึงเรื่องนี้ไว้ใน T249840: ส่วนประกอบ UI ของ Vue.js ที่ใช้ร่วมกันควรอยู่ที่ใดสำหรับโปรเจ็กต์ WMDE และ WMF ?
9 เม.ย. 2020, 16:32 น.
• Niedzielskiได้กล่าวถึงเรื่องนี้ไว้ใน T250016: อนุญาตให้ใช้ jQuery เวอร์ชันใหม่ได้เฉพาะในกรณีพิเศษสำหรับฟังก์ชันการค้นหาและ Vector ของ Vue.jsเท่านั้น
11 เม.ย. 2020, 19:40 น.
• Niedzielskiได้กล่าวถึงเรื่องนี้ไว้ใน T249304: การกำหนดค่า ESLint และ stylelint สำหรับการพัฒนาการค้นหาด้วย Vue.js ก่อนที่จะดำเนินการกับทุก repository
11 เมษายน 2563 เวลา 20:35 น.
• Niedzielskiกล่าวถึงเรื่องนี้ใน T249051: [Spike 12.75 น.] โค้ดค้นหา Vue.js เฉพาะแอปพลิเคชันอยู่ที่ใด และการตัดสินใจนี้จะมีผลกระทบในระยะยาวอย่างไร ?
16 เม.ย. 2020, 22:30 น.
Nemo_bisกดติดตามแล้ว
2 กรกฎาคม 2563 เวลา 6:25 น.
เนื้อหาได้รับอนุญาตภายใต้ Creative Commons Attribution-ShareAlike (CC BY-SA) 4.0 เว้นแต่จะระบุไว้เป็นอย่างอื่น โค้ดได้รับอนุญาตภายใต้ GNU General Public License (GPL) 2.0 หรือเวอร์ชันที่ใหม่กว่า และใบอนุญาตโอเพนซอร์สอื่นๆ การใช้เว็บไซต์นี้แสดงว่าคุณยอมรับข้อกำหนดการใช้งาน นโยบายความเป็นส่วนตัว และจรรยาบรรณ · มูลนิธิวิกิมีเดีย · นโยบายความเป็นส่วนตัว · จรรยาบรรณ · ข้อกำหนดการใช้งาน · ข้อ สงวนสิทธิ์ · CC-BY-SA · GPL · เครดิตhttps://share.gemini.google/pEMGxCGf3KUsGoogle Books
 ├─ ISBN       [RAW]
 ├─ LCCN       [RAW]
 ├─ OCLC       [RAW]
 └─ printsec   [RAW]

          X
          │
          ├── Google Maps
          ├── MOVELOG
          ├── Netlify Agent Run
          ├── Network
          ├── AWS
          ├── IAM
          └── EKSShared Chat
   │
   ├── E.G.             UNKNOWN (ยังอ่าน transcript ไม่ได้)
   ├── Code#D           UNKNOWN
   ├── Code#C           UNKNOWN
   ├── Code#A           UNKNOWN
   └── "Up address log on"  UNKNOWNarthin1254@gmail.com ยินดีต้อนรับสู่บัญชี Google ใหม่ ดูวิธีใช้บัญชีให้ได้ประโยชน์สูงสุดโดยไปที่รายการตรวจสอบบัญชี Google


ที่อยู่ IP ทำงานอย่างไรบน Google
ที่อยู่ Internet Protocol (IP) ใช้เพื่อเชื่อมต่ออินเทอร์เน็ตและระบุอุปกรณ์เพื่อให้คอมพิวเตอร์ (เช่น เดสก์ท็อป อุปกรณ์เคลื่อนที่) และเซิร์ฟเวอร์สามารถสื่อสารกันได้ 

ผู้ให้บริการอินเทอร์เน็ต (เช่น บริษัทผู้ให้บริการเคเบิลทีวี โทรศัพท์ อินเทอร์เน็ตไร้สาย หรือโทรศัพท์มือถือ) จะกำหนดที่อยู่ IP ให้กับอุปกรณ์ของคุณ ซึ่งเป็นข้อกำหนดในการใช้อินเทอร์เน็ต ที่อยู่ IP คือวิธีที่คอมพิวเตอร์บนอินเทอร์เน็ตจดจํากันและกันเพื่อส่งเว็บไซต์หรือบริการให้แก่กัน เมื่อคุณเข้าชมเว็บไซต์ เช่น google.com ผู้ให้บริการอินเทอร์เน็ตจะใช้ที่อยู่ IP ของคุณเพื่อให้ google.com เปิดขึ้นในอุปกรณ์ที่คุณกําลังใช้อยู่

วิธีที่ Google ใช้ที่อยู่ IP ของคุณ
Google ใช้ที่อยู่ IP ในเกือบทุกอย่างที่เราทํา ตั้งแต่การสร้างศูนย์ข้อมูลและการเปิดให้วิศวกรสร้างผลิตภัณฑ์ต่างๆ อย่าง Search หรือ Maps ไปจนถึงการแสดงวิดีโอ YouTube ในโทรศัพท์ของคุณ

ที่อยู่ IP ช่วยให้ Google แสดงเนื้อหาที่คุณค้นหาได้ นอกจากนี้ ยังมีการใช้ที่อยู่ IP ในรูปแบบอื่นๆ ด้วย เช่น ให้ผลการค้นหาที่เกี่ยวข้องกับตำแหน่งที่คุณอยู่และเพื่อช่วยรักษาความปลอดภัยของบัญชี

วิธีแสดงผลการค้นหาในพื้นที่โดยใช้ตําแหน่งของคุณ
ที่อยู่ IP อิงจากภูมิศาสตร์คร่าวๆ เช่นเดียวกับรหัสพื้นที่ของหมายเลขโทรศัพท์ ซึ่งหมายความว่าแอปหรือเว็บไซต์ใดก็ตามที่คุณใช้ รวมถึง google.com จะประมาณพื้นที่ทั่วไปที่คุณอยู่ได้จากที่อยู่ IP ของคุณ เมื่อ Google คาดคะเนพื้นที่ทั่วไปที่คุณอยู่ Google จะให้ผลการค้นหาที่เกี่ยวข้องกับตําแหน่งที่คุณอยู่

ตัวอย่างเช่น Google จะใช้ที่อยู่ IP เพื่อให้ข้อมูลพยากรณ์อากาศของเมืองที่คุณอยู่เมื่อคุณค้นหาด้วยคำว่าสภาพอากาศ ดูข้อมูลเพิ่มเติมเกี่ยวกับวิธีจัดการตำแหน่งใน Google

สำคัญ: อินเทอร์เน็ตจะใช้งานไม่ได้หากไม่มีที่อยู่ IP เมื่อคุณใช้เว็บไซต์ แอป หรือบริการต่างๆ เช่น Google โดยปกติแล้วเว็บไซต์เหล่านี้จะตรวจหาข้อมูลบางอย่างเกี่ยวกับตำแหน่งของคุณ

วิธีรักษาบัญชีให้ปลอดภัย
เมื่อคุณลงชื่อเข้าใช้บัญชีหรือตรวจสอบอุปกรณ์ที่ลงชื่อเข้าใช้ เราจะใช้ที่อยู่ IP เพื่อประเมินพื้นที่ทั่วไปของอุปกรณ์

อย่างไรก็ตาม คุณอาจพบกิจกรรมจากตำแหน่งที่คุณไม่รู้จักดังนี้

ตำแหน่งที่ไม่รู้จัก
ในบางกรณี Google อาจคาดการณ์ตําแหน่งจากที่อยู่ IP ไม่ได้ ในกรณีนี้ คุณจะเห็นกิจกรรมที่แสดงจาก "ตําแหน่งที่ไม่รู้จัก" คุณตรวจสอบได้ว่าเป็นกิจกรรมของคุณจริงๆ หรือไม่โดยตรวจสอบรายละเอียดอื่นๆ เช่น เวลาที่ทํากิจกรรมและเบราว์เซอร์ที่ใช้ หากจําไม่ได้ว่าได้ใช้งานในเวลานั้นหรือใช้เบราวเซอร์ดังกล่าว คุณควรใช้การตรวจสอบความปลอดภัยเพื่อช่วยปกป้องบัญชี
ฉันจำตำแหน่งนี้ไม่ได้
ที่อยู่ IP มีลักษณะอย่างไร
ที่อยู่ IP จะเป็นตัวระบุ เช่น 203.0.113.42 หรือ 2001:0002:14:5:1:2:bf35:2610

หน้านี้อาจมีเนื้อหาที่แปลโดยใช้เทคโนโลยี AI การแปลโดย AI อาจมีข้อผิดพลาด
เปิดในหน้าต่างใหม่
