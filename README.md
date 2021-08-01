# Synchronous FIFO: Assertion based Verification

FIFOs or any other memory element require more detailed verification effort before it can synthesized on hardware like FPGAs/ASIC.
Here, I have presented many different assertions that can be utilized to verify a synchronous FIFO using SystemVerilog.

I encourage you to go through them and then try to write your own assertions for your design specific FIFO. **The assertions described
are not an exhastive list. I have left other check for you to add.**
Remember, there are many ways of capturing an assertion, so don't use my constructs to be the only way of wirting assertions :-)

1) Asynchronous reset assertions

``` sv
  // Reset startup check //
  // need this at the very begining of the simulation //
  property async_rst_startup;
	  @(posedge i_clk) !i_rst_n |-> ##1 (wr_ptr==0 && rd_ptr == 0 && o_empty);
  endproperty
  
  // rst check in general
  property async_rst_chk;
	  @(negedge i_rst_n) 1'b1 |-> ##1 @(posedge i_clk) (wr_ptr==0 && rd_ptr == 0 && o_empty);
  endproperty
  ```

2) Check data written at a location is the same data read when read_ptr reaches that location
``` sv
  sequence rd_detect(ptr);
    ##[0:$] (rd_en && !o_empty && (rd_ptr == ptr));
  endsequence
  
  property data_wr_rd_chk(wrPtr);
    // local variable
    integer ptr, data;
    @(posedge i_clk) disable iff(!i_rst_n)
    (wr_en && !o_full, ptr = wrPtr, data = i_data, $display($time, " wr_ptr=%h, i_fifo=%h",wr_ptr, i_data))
    |-> ##1 first_match(rd_detect(ptr), $display($time, " rd_ptr=%h, o_fifo=%h",rd_ptr, o_data)) ##0  o_data == data;
  endproperty
```
3) Rule-1: Never write to FIFO if it's Full!
```sv
  property dont_write_if_full;
    // @(posedge i_clk) disable iff(!i_rst_n) o_full |-> ##1 $stable(wr_ptr);
    // alternative way of writing the same assertion
    @(posedge i_clk) disable iff(!i_rst_n) wr_en && o_full |-> ##1 wr_ptr == $past(wr_ptr);
  endproperty
```

4) Rule-2: Never read from an Empty FIFO!
```sv
  property dont_read_if_empty;
    @(posedge i_clk) disable iff(!i_rst_n) rd_en && o_empty |-> ##1 $stable(rd_ptr);
  endproperty
```

5) On successful write, write_ptr should only increment by 1
```sv
   property inc_wr_one;
      @(posedge i_clk) disable iff(!i_rst_n) wr_en && !o_full |-> ##1 (wr_ptr-1'b1 == $past(wr_ptr));
   endproperty
```
