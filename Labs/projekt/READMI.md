### VHDL code sensor

```vhdl
library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.NUMERIC_STD.ALL;
entity sonar is
    Port ( clk : in STD_LOGIC;
           trig : out STD_LOGIC;--trigger
           echo : in STD_LOGIC := '0';
           cm1 : out unsigned(3 downto 0); --most significant bit of distance in binary form
           cm0 : out unsigned(3 downto 0));
end sonar;
architecture sonar_arch of sonar is
begin
    process(clk)
    variable count: integer range 0 to 10000;
    variable tmpcm1: unsigned(3 downto 0) := "0000";
    variable tmpcm0 : unsigned(3 downto 0) := "0000";
    variable sendTrig: std_logic := '1';
    variable echoPrev: std_logic:= '0';
    variable echoCurrent : std_logic:= '0';
    begin
        if rising_edge(clk) then
            if sendTrig = '1' then -- send 10us trig
                if count = 10 then -- 10 us trig is completed, reset tmp signals for the new calculation
                    trig <= '0';
                    sendTrig := '0';
                    count := 0;
                    tmpcm1:= "0000";
                    tmpcm0 := "0000";
                else
                    trig <= '1';
                    count := count + 1;
                end if; 
            else -- stop sending trig, do the calculation (5800 clk cycle = 1cm)(count enters here as 0)
                if echo = '1' then -- we can start calculating the width of the echo
                    if count = 5799 then--we measuered 1 cm
                        if tmpcm0 = "1001" then
                            if tmpcm1 = "1001" then
                                sendTrig := '1';--max distance to be measured(100 cm), stop measuring, re-send the trig(also update the output)
                                cm0 <= tmpcm0;
                                cm1 <= tmpcm1;
                            else
                                tmpcm1 := tmpcm1 + "0001";
                                tmpcm0 := "0000";
                            end if;
                        else
                            tmpcm0 := tmpcm0 + "0001";
                        end if;
                        count := 0;
                    else
                        count := count + 1;
                    end if;
                    if echoPrev = '1' and echoCurrent = '0' then --falling edge of echo, edit the output, start sending new trigger
                        sendTrig := '1';
                        cm0 <= tmpcm0;
                        cm1 <= tmpcm1;
                    end if;                              
                end if;                        
            end if;
            echoPrev := echoCurrent;
            echoCurrent := echo;
        end if;      
    end process; 
end sonar_arch;

### VHDL code tb_sensor

```vhdl


library IEEE;
use IEEE.STD_LOGIC_1164.ALL;
use IEEE.NUMERIC_STD.ALL;
-- Uncomment the following library declaration if using
-- arithmetic functions with Signed or Unsigned values
--use IEEE.NUMERIC_STD.ALL;

-- Uncomment the following library declaration if instantiating
-- any Xilinx leaf cells in this code.
--library UNISIM;
--use UNISIM.VComponents.all;

entity tb_sonar is
--  Port ( );
end tb_sonar;
architecture Behavioral of tb_sonar is
    constant c_CLK_100MHZ_PERIOD : time := 10 ns;

    signal s_clk : std_logic;
    signal s_trig : std_logic;
    signal s_echo : std_logic := '0';
    signal s_cm1 : unsigned(3 downto 0);
    signal s_cm0 : unsigned(3 downto 0);
begin
    uut_ce : entity work.sonar
        port map(
        clk => s_clk,
        trig => s_trig,
        echo => s_echo,
        cm1 => s_cm1,
        cm0 => s_cm0
        );
    p_clk_gen : process
    begin
     while now < 10000 ns loop   -- 10 usec of simulation
            s_clk <= '0';
            wait for c_CLK_100MHZ_PERIOD / 2;
            s_clk <= '1';
            wait for c_CLK_100MHZ_PERIOD / 2;
        end loop;
     wait;   
    end process p_clk_gen; 
end Behavioral;
```
### Diagram
![waveforms](Images/prubeh.png)
