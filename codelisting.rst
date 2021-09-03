
Code Listing
============

Here is the full listing of the Linkam class code.

Note that ``boxedtext`` is a tool used in BMM's profile for displaying
textual information on screen.  See
https://github.com/NSLS-II-BMM/profile_collection/blob/master/startup/BMM/functions.py#L144

.. code-block:: python
   :linenos:

        from ophyd import Component as Cpt, EpicsSignal, EpicsSignalRO, PVPositioner
        from ophyd.signal import DerivedSignal
        
        from BMM.functions      import boxedtext
        
        class AtSetpoint(DerivedSignal):
            '''A signal that does bit-wise arithmetic on the Linkam's status code'''
            def __init__(self, parent_attr, *, parent=None, **kwargs):
                code_signal = getattr(parent, parent_attr)
                super().__init__(derived_from=code_signal, parent=parent, **kwargs)
        
            def inverse(self, value):
                if int(value) & 2 == 2:
                    return 1
                else:
                    return 0
        
            def forward(self, value):
                return value
        
        class Linkam(PVPositioner):
            '''An ophyd wrapper around the Linkam T96 controller
            '''
        
            ## following https://blueskyproject.io/ophyd/positioners.html#pvpositioner
            readback = Cpt(EpicsSignalRO, 'TEMP')
            setpoint = Cpt(EpicsSignal, 'SETPOINT:SET')
            status_code = Cpt(EpicsSignal, 'STATUS')
            done = Cpt(AtSetpoint, parent_attr = 'status_code')
        
            ## all the rest of the Linkam signals
            init = Cpt(EpicsSignal, 'INIT')
            model_array = Cpt(EpicsSignal, 'MODEL')
            serial_array = Cpt(EpicsSignal, 'SERIAL')
            stage_model_array = Cpt(EpicsSignal, 'STAGE:MODEL')
            stage_serial_array = Cpt(EpicsSignal, 'STAGE:SERIAL')
            firm_ver = Cpt(EpicsSignal, 'FIRM:VER')
            hard_ver = Cpt(EpicsSignal, 'HARD:VER')
            ctrllr_err = Cpt(EpicsSignal, 'CTRLLR:ERR')
            config = Cpt(EpicsSignal, 'CONFIG')
            stage_config = Cpt(EpicsSignal, 'STAGE:CONFIG')
            disable = Cpt(EpicsSignal, 'DISABLE')
            dsc = Cpt(EpicsSignal, 'DSC')
            RR_set = Cpt(EpicsSignal, 'RAMPRATE:SET')
            RR = Cpt(EpicsSignal, 'RAMPRATE')
            ramptime = Cpt(EpicsSignal, 'RAMPTIME')
            startheat = Cpt(EpicsSignal, 'STARTHEAT')
            holdtime_set = Cpt(EpicsSignal, 'HOLDTIME:SET')
            holdtime = Cpt(EpicsSignal, 'HOLDTIME')
            power = Cpt(EpicsSignalRO, 'POWER')
            lnp_speed = Cpt(EpicsSignal, 'LNP_SPEED')
            lnp_mode_set = Cpt(EpicsSignal, 'LNP_MODE:SET')
            lnp_speed_set = Cpt(EpicsSignal, 'LNP_SPEED:SET')
        
            def on(self):
                self.startheat.put(1)
        
            def off(self):
                self.startheat.put(0)
            
            def on_plan(self):
                return(yield from mv(self.startheat, 1))
        
            def off_plan(self):
                return(yield from mv(self.startheat, 0))
        
            def arr2word(self, lst):
                word = ''
                for l in lst[:-1]:
                    word += chr(l)
                return word
                
            @property
            def serial(self):
                return self.arr2word(self.serial_array.get())
        
            @property
            def model(self):
                return self.arr2word(self.model_array.get())
            
            @property
            def stage_model(self):
                return self.arr2word(self.stage_model_array.get())
            
            @property
            def stage_serial(self):
        
            @property
            def firmware_version(self):
                return self.arr2word(self.firm_ver.get())
        
            @property
            def hardware_version(self):
                return self.arr2word(self.hard_ver.get())
        
            def status(self):
                text = f'\nCurrent temperature = {self.readback.get():.1f}, setpoint = {self        .setpoint.get():.1f}\n\n'
                code = int(self.status_code.get())
                if code & 1:
                    text += error_msg('Error        : yes') + '\n'
                else:
                    text += 'Error        : no\n'
                if code & 2:
                    text += go_msg('At setpoint  : yes') + '\n'
                else:
                    text += 'At setpoint  : no\n'
                if code & 4:
                    text += go_msg('Heater       : on') + '\n'
                else:
                    text += 'Heater       : off\n'
                if code & 8:
                    text += go_msg('Pump         : on') + '\n'
                else:
                    text += 'Pump         : off\n'
                if code & 16:
                    text += go_msg('Pump Auto    : yes') + '\n'
                else:
                    text += 'Pump Auto    : no\n'
        
                boxedtext(f'Linkam {self.model}, stage {self.stage_model}', text, 'brown', width = 45)
