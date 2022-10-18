TARGET=circuit

print: $(TARGET).r1cs $(TARGET).sym
	npx snarkjs r1cs print $?

info: $(TARGET).r1cs
	npx snarkjs r1cs info $?

%.wasm %.sym %.r1cs: %.circom
	npx circom $? --r1cs --wasm --sym -v


ptn:  ptau0.ptau
ptc1: ptau1.ptau
ptc2: ptau2.ptau
ptc3: ptau3.ptau
beacon: ptau-beacon.ptau
prepare: ptau-final.ptau

verify-ptau3: ptau3.ptau
	npx snarkjs powersoftau verify $<
verify-prepare: ptau-final.ptau
	npx snarkjs powersoftau verify $<
verify-zkey: circuit3.zkey
	npx snarkjs zkey verify $(TARGET).r1cs ptau-final.ptau $<
verify-zkey-beacon: circuit-final.zkey
	npx snarkjs zkey verify $(TARGET).r1cs ptau-final.ptau $<

ptau0.ptau:
	npx snarkjs ptn bn128 12 ptau0.ptau -v

ptau1.ptau: ptau0.ptau
	npx snarkjs ptc $< $@
ptau2.ptau: ptau1.ptau
	npx snarkjs ptc $< $@
ptau3.ptau: ptau2.ptau
	npx snarkjs ptc $< $@
ptau-beacon.ptau: ptau3.ptau
	npx snarkjs ptb $< $@ 5891b5b522d5df086d0ff0b110fbd9d21bb4fc7163af34d08286a2e846f6be03 10
ptau-final.ptau: ptau-beacon.ptau
	npx snarkjs pt2 $< $@

circuit.zkey: $(TARGET).r1cs ptau-final.ptau
	#npx snarkjs zkey new $? $@
	npx snarkjs groth16 setup $? $@

circuit1.zkey: circuit.zkey
	npx snarkjs zkey contribute  $< $@
circuit2.zkey: circuit1.zkey
	npx snarkjs zkey contribute  $< $@
circuit3.zkey: circuit2.zkey
	npx snarkjs zkey contribute  $< $@
circuit-final.zkey: circuit3.zkey
	npx snarkjs zkey beacon $< $@ 15509d1dad4fb28483c79e705dd6790ef07bd82aca88d4376db41edf9ce26f25 10

verification_key.json: circuit-final.zkey
	npx snarkjs zkey export verificationkey $< $@

input.json:
	echo '{"a":3,"b":7}' > $@

witness.wtns: $(TARGET).wasm circuit-final.zkey input.json
	#npx snarkjs wtns calculate $(TARGET).wasm input.json witness.wtns
	npx snarkjs wtns debug $(TARGET).wasm input.json witness.wtns $(TARGET).sym --trigger -set -get

proof.json public.json: circuit-final.zkey witness.wtns
	npx snarkjs groth16 prove circuit-final.zkey witness.wtns proof.json public.json

verify: verification_key.json proof.json public.json
	npx snarkjs groth16 verify verification_key.json public.json proof.json

verifier.sol: circuit-final.zkey
	npx snarkjs zkey export solidityverifier $< $@


clean:
	rm -f $(TARGET).wasm $(TARGET).r1cs $(TARGET).sym ptau*.ptau circuit*.zkey verification_key.json input.json witness.wtns proof.json public.json verifier.sol

.PHONY: print info ptau verify clean


