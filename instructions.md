# Mercury Instructions Documentation

### Opcode Table
<table>
<thead>
    <tr>
        <th></th>
        <th><code>-0</code></th>
        <th><code>-1</code></th>
        <th><code>-2</code></th>
        <th><code>-3</code></th>
        <th><code>-4</code></th>
        <th><code>-5</code></th>
        <th><code>-6</code></th>
        <th><code>-7</code></th>
        <th><code>-8</code></th>
        <th><code>-9</code></th>
        <th><code>-A</code></th>
        <th><code>-B</code></th>
        <th><code>-C</code></th>
        <th><code>-D</code></th>
        <th><code>-E</code></th>
        <th><code>-F</code></th>
    </tr>
</thead>
<tbody>
    <tr>
        <th><code>0-</code></th>
        <td><a href="#rst">RST</a></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td><a href="#jcc">Jcc M0</a></td>
        <td><a href="#jcc">Jcc M1</a></td>
        <td><a href="#jcc">Jcc M2</a></td>
        <td><a href="#jcc">Jcc M3</a></td>
        <td><a href="#jcc">Jcc M4</a></td>
        <td><a href="#jcc">Jcc M5</a></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <th><code>1-</code></th>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <th><code>2-</code></th>
        <td><a href="#ret">RET</a></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td><a href="#call">CALL M0</a></td>
        <td><a href="#call">CALL M1</a></td>
        <td><a href="#call">CALL M2</a></td>
        <td><a href="#call">CALL M3</a></td>
        <td><a href="#call">CALL M4</a></td>
        <td><a href="#call">CALL M5</a></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <th><code>3-</code></th>
        <td><a href="#rti">RTI</a></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td>
            <a href="#int">INT imm</a><br/>
            <a href="#intv">INTV imm</a>
        </td>
        <td></td>
    </tr>
    <tr>
        <th><code>4-</code></th>
        <td><a href="#add">ADD %reg,&nbsp;imm</a></td>
        <td><a href="#add">ADD %reg,&nbsp;%reg</a></td>
        <td><a href="#adc">ADC %reg,&nbsp;imm</a></td>
        <td><a href="#adc">ADC %reg,&nbsp;%reg</a></td>
        <td></td>
        <td></td>
        <td></td>
        <td><a href="#inc">INC %reg</a></td>
        <td><a href="#and">AND %reg,&nbsp;imm</a></td>
        <td><a href="#and">AND %reg,&nbsp;%reg</a></td>
        <td><a href="#or">OR %reg,&nbsp;imm</a></td>
        <td><a href="#or">OR %reg,&nbsp;%reg</a></td>
        <td><a href="#xor">XOR %reg,&nbsp;imm</a></td>
        <td><a href="#xor">XOR %reg,&nbsp;%reg</a></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <th><code>5-</code></th>
        <td><a href="#sub">SUB imm</a></td>
        <td><a href="#sub">SUB %reg</a></td>
        <td><a href="#sbc">SBC imm</a></td>
        <td><a href="#sbc">SBC %reg</a></td>
        <td><a href="#cmp">CMP imm</a></td>
        <td><a href="#cmp">CMP %reg</a></td>
        <td></td>
        <td><a href="#dec">DEC %reg</a></td>
        <td><a href="#nand">NAND imm</a></td>
        <td><a href="#nand">NAND %reg</a></td>
        <td><a href="#nor">NOR imm</a></td>
        <td><a href="#nor">NOR %reg</a></td>
        <td><a href="#xnor">XNOR imm</a></td>
        <td><a href="#xnor">XNOR %reg</a></td>
        <td></td>
        <td><a href="#not">NOT %reg</a></td>
    </tr>
    <tr>
        <th><code>6-</code></th>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td><a href="#lsl">LSL %reg</a></td>
        <td></td>
        <td><a href="#rol">ROL %reg</a></td>
        <td></td>
        <td></td>
        <td></td>
        <td><a href="#swp">SWP %reg</a></td>
    </tr>
    <tr>
        <th><code>7-</code></th>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td><a href="#lsr">LSL %reg</a></td>
        <td></td>
        <td><a href="#ror">ROL %reg</a></td>
        <td></td>
        <td></td>
        <td></td>
        <td><a href="#swp">SWPL %reg</a></td>
    </tr>
    <tr>
        <th><code>8-</code></th>
        <td><a href="#mov">MOV [M0],&nbsp;%reg</a></td>
        <td><a href="#mov">MOV [M1],&nbsp;%reg</a></td>
        <td><a href="#mov">MOV [M2],&nbsp;%reg</a></td>
        <td><a href="#mov">MOV [M3],&nbsp;%reg</a></td>
        <td><a href="#mov">MOV [M4],&nbsp;%reg</a></td>
        <td><a href="#mov">MOV [M5],&nbsp;%reg</a></td>
        <td></td>
        <td><a href="#mov">MOV %reg,&nbsp;%reg</a></td>
        <td><a href="#phm">PHM.xx [M0]</a></td>
        <td><a href="#phm">PHM.xx [M1]</a></td>
        <td><a href="#phm">PHM.xx [M2]</a></td>
        <td><a href="#phm">PHM.xx [M3]</a></td>
        <td><a href="#phm">PHM.xx [M4]</a></td>
        <td><a href="#phm">PHM.xx [M5]</a></td>
        <td><a href="#phi">PHI.xx imm</a></td>
        <td><a href="#psh">PSH.xx %reg</a></td>
    </tr>
    <tr>
        <th><code>9-</code></th>
        <td><a href="#mov">MOVL [M0],&nbsp;%reg</a></td>
        <td><a href="#mov">MOVL [M1],&nbsp;%reg</a></td>
        <td><a href="#mov">MOVL [M2],&nbsp;%reg</a></td>
        <td><a href="#mov">MOVL [M3],&nbsp;%reg</a></td>
        <td><a href="#mov">MOVL [M4],&nbsp;%reg</a></td>
        <td><a href="#mov">MOVL [M5],&nbsp;%reg</a></td>
        <td></td>
        <td><a href="#mov">MOVL %reg,&nbsp;%reg</a></td>
        <td><a href="#phm">PHML.xx [M0]</a></td>
        <td><a href="#phm">PHML.xx [M1]</a></td>
        <td><a href="#phm">PHML.xx [M2]</a></td>
        <td><a href="#phm">PHML.xx [M3]</a></td>
        <td><a href="#phm">PHML.xx [M4]</a></td>
        <td><a href="#phm">PHML.xx [M5]</a></td>
        <td><a href="#phi">PHIL.xx imm</a></td>
        <td><a href="#psh">PSHL.xx %reg</a></td>
    </tr>
    <tr>
        <th><code>A-</code></th>
        <td><a href="#mvi">MVI [M0],&nbsp;imm</a></td>
        <td><a href="#mvi">MVI [M1],&nbsp;imm</a></td>
        <td><a href="#mvi">MVI [M2],&nbsp;imm</a></td>
        <td><a href="#mvi">MVI [M3],&nbsp;imm</a></td>
        <td><a href="#mvi">MVI [M4],&nbsp;imm</a></td>
        <td><a href="#mvi">MVI [M5],&nbsp;imm</a></td>
        <td></td>
        <td><a href="#mvi">MVI %reg,&nbsp;imm</a></td>
        <td><a href="#ppm">PPM.xx [M0]</a></td>
        <td><a href="#ppm">PPM.xx [M1]</a></td>
        <td><a href="#ppm">PPM.xx [M2]</a></td>
        <td><a href="#ppm">PPM.xx [M3]</a></td>
        <td><a href="#ppm">PPM.xx [M4]</a></td>
        <td><a href="#ppm">PPM.xx [M5]</a></td>
        <td></td>
        <td><a href="#pop">POP.xx %reg</a></td>
    </tr>
    <tr>
        <th><code>B-</code></th>
        <td><a href="#mvi">MVIL [M0],&nbsp;imm</a></td>
        <td><a href="#mvi">MVIL [M1],&nbsp;imm</a></td>
        <td><a href="#mvi">MVIL [M2],&nbsp;imm</a></td>
        <td><a href="#mvi">MVIL [M3],&nbsp;imm</a></td>
        <td><a href="#mvi">MVIL [M4],&nbsp;imm</a></td>
        <td><a href="#mvi">MVIL [M5],&nbsp;imm</a></td>
        <td></td>
        <td><a href="#mvi">MVIL %reg,&nbsp;imm</a></td>
        <td><a href="#ppm">PPML.xx [M0]</a></td>
        <td><a href="#ppm">PPML.xx [M1]</a></td>
        <td><a href="#ppm">PPML.xx [M2]</a></td>
        <td><a href="#ppm">PPML.xx [M3]</a></td>
        <td><a href="#ppm">PPML.xx [M4]</a></td>
        <td><a href="#ppm">PPML.xx [M5]</a></td>
        <td></td>
        <td><a href="#pop">POPL.xx %reg</a></td>
    </tr>
    <tr>
        <th><code>C-</code></th>
        <td><a href="#mvm">MVM reg,&nbsp;[M0]</a></td>
        <td><a href="#mvm">MVM reg,&nbsp;[M1]</a></td>
        <td><a href="#mvm">MVM reg,&nbsp;[M2]</a></td>
        <td><a href="#mvm">MVM reg,&nbsp;[M3]</a></td>
        <td><a href="#mvm">MVM reg,&nbsp;[M4]</a></td>
        <td><a href="#mvm">MVM reg,&nbsp;[M5]</a></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <th><code>D-</code></th>
        <td><a href="#mvm">MVML reg,&nbsp;[M0]</a></td>
        <td><a href="#mvm">MVML reg,&nbsp;[M1]</a></td>
        <td><a href="#mvm">MVML reg,&nbsp;[M2]</a></td>
        <td><a href="#mvm">MVML reg,&nbsp;[M3]</a></td>
        <td><a href="#mvm">MVML reg,&nbsp;[M4]</a></td>
        <td><a href="#mvm">MVML reg,&nbsp;[M5]</a></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
    </tr>
    <tr>
        <th><code>E-</code></th>
        <td><a href="#mvm">MVM S0</a></td>
        <td><a href="#mvm">MVM S1</a></td>
        <td><a href="#mvm">MVM S2</a></td>
        <td><a href="#mvm">MVM S3</a></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td><a href="#nop">NOP</a></td>
    </tr>
    <tr>
        <th><code>F-</code></th>
        <td><a href="#mvm">MVML S0</a></td>
        <td><a href="#mvm">MVML S1</a></td>
        <td><a href="#mvm">MVML S2</a></td>
        <td><a href="#mvm">MVML S3</a></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td></td>
        <td><a href="#hlt">HLT</a></td>
    </tr>
</tbody>
</table>

#### Memory Addressing Modes
|    | Meaning                          |
|:--:|:---------------------------------|
| M0 | `%reg16`                         |
| M1 | `%reg32`                         |
| M2 | `%reg16+imm16`                   |
| M3 | `%reg32+imm16`                   |
| M4 | `%reg16+imm32`                   |
| M5 | `%reg32+imm32`                   |
| S0 | `[%reg16+imm32], [%reg16+imm32]` |
| S1 | `[%reg16+imm32], [%reg32+imm32]` |
| S2 | `[%reg32+imm32], [%reg16+imm32]` |
| S3 | `[%reg32+imm32], [%reg32+imm32]` |

## Instruction List

## Instructions