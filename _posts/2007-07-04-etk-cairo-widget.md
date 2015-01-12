---
layout: post
title: Etk Cairo Widget
date: '2007-07-04T23:17:00-04:00'
tags:
- Enlightenment
- etk
- enlightenment
- cairo
tumblr_url: http://www.hisham.cc/post/30203507188/etk-cairo-widget
---
So someone on irc (bmz from #etk) mentioned he wanted to do some drawings using the EFL and would like to be able to draw shapes and curves onto Etk widgets. After a couple of hours of tinkering about, I created the Etk_Cairo widget. This widget allows you to creae a “surface” on which you can use Cairo to draw. A short example would be like this.


static void _etk_cairo_test_redraw_required_cb(Etk_Object *object, void *data)
{
  Etk_Widget *cairo;
  cairo_t *cr;
  int w, h;
    
  if (!(cairo = ETK_WIDGET(object)))
    return;
    
  etk_widget_geometry_get(cairo, NULL, NULL, &w, &h);
  cr = etk_cairo_get(ETK_CAIRO(cairo));
  _etk_cairo_test_draw(cr, 0, 0, w, h);
}
      
int main(int argc, char **argv)
{
  Etk_Widget *window;
  Etk_Widget *cairo;
  Etk_Widget *frame;
      
  etk_init(&argc, &argv);
      
  window = etk_window_new();
  etk_window_title_set(ETK_WINDOW(window), "Etk-Cairo Test");
      
  cairo = etk_cairo_new(WIDTH, HEIGHT);
  etk_signal_connect("redraw-required", ETK_OBJECT(cairo),
    ETK_CALLBACK(_etk_cairo_test_redraw_required_cb), NULL);

  frame = etk_frame_new("Cairo Drawing");
  etk_container_add(ETK_CONTAINER(frame), cairo);
  
  etk_container_add(ETK_CONTAINER(window), frame);
  etk_window_resize(ETK_WINDOW(window), WIDTH, HEIGHT);
  etk_widget_show_all(window);

  etk_main();
  etk_shutdown();

  return 0;
}



In this example, the _etk_cairo_test_draw function does all the cairo stuff and can result in something like this or this.
