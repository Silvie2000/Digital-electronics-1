# Digital-electronics-1
## Dekodér
**Dekodér binárního kódu na 1z8.**

```vhdl
library IEEE;
use IEEE.STD_LOGIC_1164.ALL, IEEE.NUMERIC_STD.all;
use IEEE.STD_LOGIC_ARITH.ALL;
use IEEE.STD_LOGIC_UNSIGNED.ALL;
 
entity Decoder is
  port (
    A : in std_logic_vector (2 downto 0);
    Y : out std_logic_vector(7 downto 0)
  );
end Decoder;
 
architecture behavioral of Decoder is
begin
  process (A) begin
    case A is
      when "000" => Y <= "00000001";
      when "001" => Y <= "00000010";
      when "010" => Y <= "00000100";
      when "011" => Y <= "00001000";
      when "100" => Y <= "00010000";
      when "101" => Y <= "00100000";
      when "110" => Y <= "01000000";
      when "111" => Y <= "10000000";
      when others => Y <= "00000000";
    end case;
  end process;
end behavioral;
```
