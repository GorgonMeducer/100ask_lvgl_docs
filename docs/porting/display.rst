.. _display_interface:

============================
Display interface（显示接口）
============================

.. raw:: html

   <details>
     <summary>显示原文</summary>

To create a display for LVGL call
:cpp:expr:`lv_display_t * display = lv_display_create(hor_res, ver_res)`. You can create
a multiple displays and a different driver for each (see below),

.. raw:: html

   </details>
   <br>


要为 LVGL 创建显示，请调用 :cpp:expr:`lv_display_t * display = lv_display_create(hor_res, ver_res)`。您可以创建多个显示器和每个显示器的不同驱动程序（见下文），


Basic setup（基本设置）
**********************

.. raw:: html

   <details>
     <summary>显示原文</summary>

Draw buffer(s) are simple array(s) that LVGL uses to render the screen's
content. Once rendering is ready the content of the draw buffer is sent
to the display using the ``flush_cb`` function.

.. raw:: html

   </details>
   <br>


绘制缓冲区是 LVGL 用来渲染屏幕的简单数组 内容。渲染准备就绪后，将发送绘制缓冲区的内容使用 ``flush_cb`` 功能显示。


flush_cb
--------

.. raw:: html

   <details>
     <summary>显示原文</summary>

An example ``flush_cb`` looks like this:

.. code:: c

   void my_flush_cb(lv_display_t * display, const lv_area_t * area, void * px_map)
   {
       /*The most simple case (but also the slowest) to put all pixels to the screen one-by-one
        *`put_px` is just an example, it needs to be implemented by you.*/
       uint16_t * buf16 = (uint16_t *)px_map; /*Let's say it's a 16 bit (RGB565) display*/
       int32_t x, y;
       for(y = area->y1; y <= area->y2; y++) {
           for(x = area->x1; x <= area->x2; x++) {
               put_px(x, y, *buf16);
               buf16++;
           }
       }

       /* IMPORTANT!!!
        * Inform LVGL that you are ready with the flushing and buf is not used anymore*/
       lv_display_flush_ready(disp);
   }

Use :cpp:expr:`lv_display_set_flush_cb(disp, my_flush_cb)` to set a new ``flush_cb``.

:cpp:expr:`lv_display_flush_ready(disp)` needs to be called when flushing is ready
to inform LVGL the buffer is not used anymore by the driver and it can
render new content into it.

LVGL might render the screen in multiple chunks and therefore call
``flush_cb`` multiple times. To see if the current one is the last chunk
of rendering use :cpp:expr:`lv_display_flush_is_last(display)`.

.. raw:: html

   </details>
   <br>


示例 ``flush_cb`` 如下所示：

.. code:: c

   void my_flush_cb(lv_display_t * display, const lv_area_t * area, void * px_map)
   {
       /*The most simple case (but also the slowest) to put all pixels to the screen one-by-one
        *`put_px` is just an example, it needs to be implemented by you.*/
       uint16_t * buf16 = (uint16_t *)px_map; /*Let's say it's a 16 bit (RGB565) display*/
       int32_t x, y;
       for(y = area->y1; y <= area->y2; y++) {
           for(x = area->x1; x <= area->x2; x++) {
               put_px(x, y, *buf16);
               buf16++;
           }
       }

       /* IMPORTANT!!!
        * Inform LVGL that you are ready with the flushing and buf is not used anymore*/
       lv_display_flush_ready(disp);
   }

使用 :cpp:expr:`lv_display_set_flush_cb(disp, my_flush_cb)` 设置新的 ``flush_cb``。

:cpp:expr:`lv_display_flush_ready(disp)` 需要在冲洗准备就绪时调用通知 LVGL 缓冲区不再被驱动程序使用，它可以将新内容渲染到其中。

LVGL 可能会在多个块中呈现屏幕，因此会多次调用 ``flush_cb``。查看当前块是否是最后一个块 渲染使用的 :cpp:expr:`lv_display_flush_is_last(display)`。


Draw buffers（绘制缓冲区）
-------------------------

.. raw:: html

   <details>
     <summary>显示原文</summary>

The draw buffers can be set with :cpp:expr:`lv_display_set_buffers(display, buf1, buf2, buf_size_byte, render_mode)`

-  ``buf1`` a buffer where LVGL can render
-  ``buf2`` a second optional buffer (see more details below)
-  ``buf_size_byte`` size of the buffer(s) in bytes
-  ``render_mode``

   -  :cpp:enumerator:`LV_DISPLAY_RENDER_MODE_PARTIAL` Use the buffer(s) to render the
      screen in smaller parts. This way the buffers can be smaller then
      the display to save RAM. At least 1/10 screen size buffer(s) are
      recommended. In ``flush_cb`` the rendered images needs to be
      copied to the given area of the display. In this mode if a button is pressed
      only the button's area will be redrawn.
   -  :cpp:enumerator:`LV_DISPLAY_RENDER_MODE_DIRECT` The buffer(s) has to be screen
      sized and LVGL will render into the correct location of the
      buffer. This way the buffer always contain the whole image. If two
      buffer are used the rendered areas are automatically copied to the
      other buffer after flushing. Due to this in ``flush_cb`` typically
      only a frame buffer address needs to be changed. If a button is pressed
      only the button's area will be redrawn.
   -  :cpp:enumerator:`LV_DISPLAY_RENDER_MODE_FULL` The buffer(s) has to be screen
      sized and LVGL will always redraw the whole screen even if only 1
      pixel has been changed. If two screen sized draw buffers are
      provided, LVGL's display handling works like "traditional" double
      buffering. This means the ``flush_cb`` callback only has to update
      the address of the frame buffer to the ``px_map`` parameter.

Example:

.. raw:: html

   </details>
   <br>


绘制缓冲区可以用 :cpp:expr:`lv_display_set_buffers(display, buf1, buf2, buf_size_byte, render_mode)`进行设置

-  ``buf1`` LVGL 可以渲染的缓冲区
-  ``buf2`` 第二个可选缓冲区（请参阅下面的更多详细信息）
-  ``buf_size_byte`` 缓冲区的大小（以字节为单位）
-  ``render_mode``

-  :cpp:enumerator:`LV_DISPLAY_RENDER_MODE_PARTIAL` 使用缓冲区呈现屏幕以较小的部分进行。这样，缓冲区可以更小显示用于节省 RAM。至少 1/10 的屏幕大小缓冲区是推荐。在 ``flush_cb`` 渲染图像中需要复制到显示器的给定区域。在此模式下，如果按下按钮 只有按钮的区域将被重新绘制。
-  :cpp:enumerator:`LV_DISPLAY_RENDER_MODE_DIRECT` 缓冲区必须是 screen size 和 LVGL 将渲染到缓冲区。这样，缓冲区始终包含整个图像。如果两个缓冲区被使用，渲染区域会自动复制到冲洗后的其他缓冲区。因此，由于 ``flush_cb``中的这一点，通常只需更改帧缓冲器地址。如果按下按钮 只有按钮的区域将被重新绘制。
-  :cpp:enumerator:`LV_DISPLAY_RENDER_MODE_FULL` 缓冲区必须是 screen size 和 LVGL 将始终重绘整个屏幕，即使只有 1 像素已更改。如果两个屏幕大小的绘制缓冲区 前提是LVGL的显示处理工作方式类似于“传统”双缓冲。这意味着 ``flush_cb`` 回调只需要更新帧缓冲区到 ``px_map`` 参数的地址。

例如：


.. code:: c

   static uint16_t buf[LCD_HOR_RES * LCD_VER_RES / 10];
   lv_display_set_buffers(disp, buf, NULL, sizeof(buf), LV_DISPLAY_RENDER_MODE_PARTIAL);

One buffer（一个缓冲区）
^^^^^^^^^^^^^^^^^^^^^^^

.. raw:: html

   <details>
     <summary>显示原文</summary>

If only one buffer is used LVGL draws the content of the screen into
that draw buffer and sends it to the display via the ``flush_cb``. LVGL
then needs to wait until :cpp:expr:`lv_display_flush_ready` is called
(that is the content of the buffer is sent to the
display) before drawing something new into it.

.. raw:: html

   </details>
   <br>


如果只使用一个缓冲区，LVGL 将屏幕内容绘制到该绘制缓冲区中并通过 ``flush_cb`` 将其发送到显示器。 然后需要等待，直到调用 :cpp:expr:`lv_display_flush_ready`（即缓冲区的内容被发送到显示），然后在其中绘制新内容。

Two buffers（两个缓冲区）
^^^^^^^^^^^^^^^^^^^^^^^^

.. raw:: html

   <details>
     <summary>显示原文</summary>

If two buffers are used LVGL can draw into one buffer while the content
of the other buffer is sent to the display in the background. DMA or
other hardware should be used to transfer data to the display so the MCU
can continue drawing. This way, the rendering and refreshing of the
display become parallel operations.

.. raw:: html

   </details>
   <br>


如果使用两个缓冲区，LVGL 可以绘制到一个缓冲区中，而另一个缓冲区的内容被发送到后台显示。 应使用 DMA 或其他硬件将数据传输到显示器，因此MCU可以继续绘制。 这样，显示的渲染和刷新变得并行。


Advanced options（高级选项）
***************************

Resolution（分辨率）
-------------------

.. raw:: html

   <details>
     <summary>显示原文</summary>

To set the resolution of the display after creation use
:cpp:expr:`lv_display_set_resolution(display, hor_res, ver_res)`

It's not mandatory to use the whole display for LVGL, however in some
cases the physical resolution is important. For example the touchpad
still sees the whole resolution and the values needs to be converted to
the active LVGL display area. So the physical resolution and the offset
of the active area can be set with
:cpp:expr:`lv_display_set_physical_resolution(disp, hor_res, ver_res)` and
:cpp:expr:`lv_display_set_offset(disp, x, y)`

.. raw:: html

   </details>
   <br>


要在创建后设置显示器的分辨率，请使用 :cpp:expr:`lv_display_set_resolution(display, hor_res, ver_res)`

对于 LVGL，不强制要求使用整个显示器，但在某些物理分辨率很重要。例如触摸板仍然看到整个分辨率，并且需要将值转换为活动 LVGL 显示区域。所以物理分辨率和偏移量的有效区域可以用 :cpp:expr:`lv_display_set_physical_resolution(disp, hor_res, ver_res)` 和 :cpp:expr:`lv_display_set_offset(disp, x, y)` 进行设置


Flush wait callback（刷新等待回调）
----------------------------------

.. raw:: html

   <details>
     <summary>显示原文</summary>

By using :cpp:expr:`lv_display_flush_ready` LVGL will spin in a loop
while waiting for flushing.

However with the help of :cpp:expr:`lv_display_set_flush_wait_cb` a custom
wait callback be set for flushing. This callback can use a semaphore, mutex,
or anything else to optimize while the waiting for flush.

If ``flush_wait_cb`` is not set, LVGL assume that `lv_display_flush_ready`
is used.


.. raw:: html

   </details>
   <br>


通过使用 :cpp:expr:`lv_display_flush_ready` 在等待冲洗时LVGL将在循环中旋转。

但是，借助 :cpp:expr:`lv_display_set_flush_wait_cb` 自定义 等待回调设置为刷新。此回调可以使用信号量、互斥量、 或其他任何在等待冲洗时进行优化的内容。

如果 ``flush_wait_cb`` 未设置，则 LVGL 假定使用了 `lv_display_flush_ready`。


Rotation（旋转）
---------------

.. raw:: html

   <details>
     <summary>显示原文</summary>

LVGL supports rotation of the display in 90 degree increments. You can
select whether you would like software rotation or hardware rotation.

The orientation of the display can be changed with
``lv_display_set_rotation(disp, LV_DISPLAY_ROTATION_0/90/180/270)``.
LVGL will swap the horizontal and vertical resolutions internally
according to the set degree. When changing the rotation
:cpp:expr:`LV_EVENT_SIZE_CHANGED` is sent to the display to allow
reconfiguring the hardware. In lack of hardware display rotation support
:cpp:expr:`lv_draw_sw_rotate` can be used to rotate the buffer in the
``flush_cb``.

.. raw:: html

   </details>
   <br>


LVGL 支持以 90 度为增量旋转显示器。您可以选择是要软件轮换还是硬件轮换。

可以使用 ``lv_display_set_rotation(disp, LV_DISPLAY_ROTATION_0/90/180/270)`` 更改显示器的方向。 LVGL 将在内部交换水平和垂直分辨率 根据设定的度数。更改旋转时，:cpp:expr:`LV_EVENT_SIZE_CHANGED` 被发送到显示器以允许重新配置硬件。在缺少硬件显示旋转支持的情况下，可以使用 :cpp:expr:`lv_draw_sw_rotate` 来旋转 ``flush_cb`` 中的缓冲区。


Color format（颜色格式）
-----------------------

.. raw:: html

   <details>
     <summary>显示原文</summary>

The default color format of the display is set according to :c:macro:`LV_COLOR_DEPTH`
(see ``lv_conf.h``)

- :c:macro:`LV_COLOR_DEPTH` ``32``: XRGB8888（4 字节/像素）
- :c:macro:`LV_COLOR_DEPTH` ``24``: RGB888（3 字节/像素）
- :c:macro:`LV_COLOR_DEPTH` ``16``: RGB565（2字节/像素）
- :c:macro:`LV_COLOR_DEPTH` ``8``:L8（1 字节/像素）尚不支持

The ``color_format`` can be changed with
:cpp:expr:`lv_display_set_color_depth(display, LV_COLOR_FORMAT_...)`.
Besides the default value :c:macro:`LV_COLOR_FORMAT_ARGB8888` can be
used as a well.

It's very important that draw buffer(s) should be large enough for any
selected color format.


.. raw:: html

   </details>
   <br>


显示器的默认颜色格式是根据 :c:macro:`LV_COLOR_DEPTH` (请参阅 ``lv_conf.h``)

- :c:macro:`LV_COLOR_DEPTH` ``32``: XRGB8888 (4 bytes/pixel)
- :c:macro:`LV_COLOR_DEPTH` ``24``: RGB888 (3 bytes/pixel)
- :c:macro:`LV_COLOR_DEPTH` ``16``: RGB565 (2 bytes/pixel)
- :c:macro:`LV_COLOR_DEPTH` ``8``: L8 (1 bytes/pixel) Not supported yet

可以用 :cpp:expr:`lv_display_set_color_depth(display, LV_COLOR_FORMAT_...)`。 此外，默认值 :c:macro:`LV_COLOR_FORMAT_ARGB8888` 可以是用作井。

绘制缓冲区应足够大，以便于任何选定的颜色格式。


Swap endianness（交换字节序）
----------------------------

.. raw:: html

   <details>
     <summary>显示原文</summary>

In case of RGB565 color format it might be required to swap the 2 bytes
because the SPI, I2C or 8 bit parallel port periphery sends them in the wrong order.

The ideal solution is configure the hardware to handle the 16 bit data with different byte order,
however if it's not possible :cpp:expr:`lv_draw_sw_rgb565_swap(buf, buf_size_in_px)`
can be called in the ``flush_cb`` to swap the bytes.

If you wish you can also write your own function, or use assembly instructions for
the fastest possible byte swapping.

Note that this is not about swapping the Red and Blue channel but converting

``RRRRR GGG | GGG BBBBB``

to

``GGG BBBBB | RRRRR GGG``.


.. raw:: html

   </details>
   <br>


如果是 RGB565 颜色格式，则可能需要交换 2 个字节 因为 SPI、I2C 或 8 位并行端口外设以错误的顺序发送它们。

理想的解决方案是配置硬件以处理具有不同字节顺序的 16 位数据， 但是，如果不可能，可以在 ``flush_cb`` 中调用 :cpp:expr:`lv_draw_sw_rgb565_swap(buf, buf_size_in_px)` 来交换字节。

如果你愿意，你也可以编写自己的函数，或者使用汇编指令来 尽可能快的字节交换。

请注意，这不是交换红色和蓝色通道，而是转换


``RRRRR GGG | GGG BBBBB``

到

``GGG BBBBB | RRRRR GGG``.


User data（用户数据）
--------------------

.. raw:: html

   <details>
     <summary>显示原文</summary>

With :cpp:expr:`lv_display_set_user_data(disp, p)` a pointer to a custom data can
be stored in display object.

.. raw:: html

   </details>
   <br>


使用 :cpp:expr:`lv_display_set_user_data(disp, p)` 指向自定义数据的指针可以 存储在显示对象中。


Decoupling the display refresh timer
------------------------------------

.. raw:: html

   <details>
     <summary>显示原文</summary>


Normally the dirty (a.k.a invalid) areas are checked and redrawn in
every :c:macro:`LV_DEF_REFR_PERIOD` milliseconds (set in ``lv_conf.h``).
However, in some cases you might need more control on when the display
refreshing happen, for example to synchronize rendering with VSYNC or
the TE signal.

You can do this in the following way:

.. code:: c

   /*Delete the original display refresh timer*/
   lv_display_delete_refr_timer(disp);

   /*Call this anywhere you want to refresh the dirty areas*/
   _lv_display_refr_timer(NULL);

If you have multiple displays call :cpp:expr:`lv_display_set_default(disp1)` to
select the display to refresh before :cpp:expr:`_lv_display_refr_timer(NULL)`.


.. note:: that :cpp:func:`lv_timer_handler` and :cpp:func:`_lv_display_refr_timer` can not  run at the same time.


If the performance monitor is enabled, the value of :c:macro:`LV_DEF_REFR_PERIOD` needs to be set to be
consistent with the refresh period of the display to ensure that the statistical results are correct.


.. raw:: html

   </details>
   <br>


通常，脏区（又称无效区）会被检查并重新绘制每 :c:macro:`LV_DEF_REFR_PERIOD` 毫秒（设置在 ``lv_conf.h`` 中）。 但是，在某些情况下，您可能需要对显示时间进行更多控制刷新发生，例如将渲染与 VSYNC 同步或 TE 信号。

您可以通过以下方式执行此操作：  

.. code:: c

   /*Delete the original display refresh timer*/
   lv_timer_delete(disp->refr_timer);
   disp->refr_timer = NULL;


   /*Call this anywhere you want to refresh the dirty areas*/
   _lv_display_refr_timer(NULL);

如果您有多个显示器，请调用 :cpp:expr:`lv_display_set_default(disp1)` 在 :cpp:expr:`_lv_display_refr_timer(NULL)` 之前选择要刷新的显示器。


.. 注意::  :cpp:func:`lv_timer_handler` 和 :cpp:func:`_lv_display_refr_timer` 不能同时运行。.


如果启用了性能监视器，则需要将 :c:macro:`LV_DEF_REFR_PERIOD` 的值设置为与显示器的刷新周期一致，以确保统计结果正确无误。


Force refreshing（强制刷新）
---------------------------

.. raw:: html

   <details>
     <summary>显示原文</summary>

Normally the invalidated areas (marked for redraw) are rendered in :cpp:func:`lv_timer_handler` in every
:cpp:macro:`LV_DEF_REFR_PERIOD`milliseconds. However, by using :cpp:func:`lv_refr_now(display)` you can ask LVGL to
redraw the invalid areas immediately. The refreshing will happen in :cpp:func:`lv_refr_now` which might take
longer time.

The parameter of :cpp:func:`lv_refr_now` is a display to refresh. If ``NULL`` is set the default display will be updated.

.. raw:: html

   </details>
   <br>


通常，无效区域（标记为重绘）在每个中的 :cpp:func:`lv_timer_handler` 中呈现 :cpp:macro:`LV_DEF_REFR_PERIOD` 毫秒。但是，通过使用 :cpp:func:`lv_refr_now（display）`，您可以要求LVGL立即重新绘制无效区域。刷新将发生在 :cpp:func:`lv_refr_now` 中，这可能需要更长的时间。

:cpp:func:`lv_refr_now` 的参数是要刷新的显示。如果设置了 ``NULL`` ，则将更新默认显示。


Events（事件）
*************

.. raw:: html

   <details>
     <summary>显示原文</summary>

:cpp:expr:`lv_display_add_event_cb(disp, event_cb, LV_EVENT_..., user_data)` adds
an event handler to a display. The following events are sent:

- :cpp:enumerator:`LV_EVENT_INVALIDATE_AREA` An area is invalidated (marked for redraw).
  :cpp:expr:`lv_event_get_param(e)` returns a pointer to an :cpp:struct:`lv_area_t`
  variable with the coordinates of the area to be invalidated. The area can
  be freely modified if needed to adopt it the special requirement of the
  display. Usually needed with monochrome displays to invalidate ``N x 8``
  rows or columns at once.
- :cpp:enumerator:`LV_EVENT_REFR_REQUEST`: Sent when something happened that requires redraw.
- :cpp:enumerator:`LV_EVENT_REFR_START`: Sent when a refreshing cycle starts. Sent even if there is nothing to redraw.
- :cpp:enumerator:`LV_EVENT_REFR_READY`: Sent when refreshing is ready (after rendering and calling the ``flush_cb``). Sent even if no redraw happened.
- :cpp:enumerator:`LV_EVENT_RENDER_START`: Sent when rendering starts.
- :cpp:enumerator:`LV_EVENT_RENDER_READY`: Sent when rendering is ready (before calling the ``flush_cb``)
- :cpp:enumerator:`LV_EVENT_FLUSH_START`: Sent before the ``flush_cb`` is called.
- :cpp:enumerator:`LV_EVENT_FLUSH_READY`: Sent when the ``flush_cb`` returned.
- :cpp:enumerator:`LV_EVENT_RESOLUTION_CHANGED`: Sent when the resolution changes due
  to :cpp:func:`lv_display_set_resolution` or :cpp:func:`lv_display_set_rotation`.


.. raw:: html

   </details>
   <br>


:cpp:expr:`lv_display_add_event_cb(disp, event_cb, LV_EVENT_..., user_data)` 添加显示的事件处理程序。将发送以下事件：

- :cpp:enumerator:`LV_EVENT_INVALIDATE_AREA` 区域无效（标记为重绘）。:cpp:expr:`lv_event_get_param(e)` 返回一个指向 :cpp:struct:`lv_area_t` 的指针变量，该变量具有无效的区域的坐标。该区域如果需要，可以自由修改以采用它的特殊要求显示。通常需要与单色显示器一起使用，以一次使 ``N x 8`` 行或列失效。
- :cpp:enumerator:`LV_EVENT_REFR_REQUEST`: 在发生需要重绘的事情时发送。
- :cpp:enumerator:`LV_EVENT_REFR_START`: 刷新周期开始时发送。即使没有什么可重绘的，也会发送。
- :cpp:enumerator:`LV_EVENT_REFR_READY`: 在刷新准备就绪时发送（渲染并调用 ``flush_cb`` ）。即使没有发生重绘，也会发送。
- :cpp:enumerator:`LV_EVENT_RENDER_START`: 在渲染开始时发送。
- :cpp:enumerator:`LV_EVENT_RENDER_READY`: 在渲染准备就绪时发送（在调用 ``flush_cb`` ）
- :cpp:enumerator:`LV_EVENT_FLUSH_START`: 在调用 ``flush_cb`` 之前发送。
- :cpp:enumerator:`LV_EVENT_FLUSH_READY`: 在返回 ``flush_cb`` 时发送。
- :cpp:enumerator:`LV_EVENT_RESOLUTION_CHANGED`: 在决议更改到期时发送 :cpp:func:`lv_display_set_resolution` 或 :cpp:func:`lv_display_set_rotation`。


Further reading（深入学习）
**************************

.. raw:: html

   <details>
     <summary>显示原文</summary>

-  `lv_port_disp_template.c <https://github.com/lvgl/lvgl/blob/master/examples/porting/lv_port_disp_template.c>`__
   for a template for your own driver.
-  :ref:`Drawing <drawing>` to learn more about how rendering
   works in LVGL.
-  :ref:`display_features` to learn more about higher
   level display features.

.. raw:: html

   </details>
   <br>


- `lv_port_disp_template.c <https://github.com/lvgl/lvgl/blob/master/examples/porting/lv_port_disp_template.c>`__ 获取您自己的驱动程序的模板。
- :ref:`Drawing <drawing>` 了解更多关于渲染在 LVGL 中是如何工作的。
- :ref:`display_features` ，以了解有关更高级别显示功能的更多信息。

API
***
