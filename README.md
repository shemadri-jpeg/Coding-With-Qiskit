# Coding-With-Qiskit
import qiskit
qiskit.__version__
from qiskit_ibm_runtime import QiskitRuntimeService

service = QiskitRuntimeService(channel="ibm_quantum", token='7b42e152520c2d418c7d6e903b322251ac5f9c35852a78c8063c9d25e1ea64fc22bdb79954e7ca4c64164db4248d28e306c83dc755f79799392255a1b789a754')
QiskitRuntimeService.save_account(channel="ibm_quantum", token='7b42e152520c2d418c7d6e903b322251ac5f9c35852a78c8063c9d25e1ea64fc22bdb79954e7ca4c64164db4248d28e306c83dc755f79799392255a1b789a754', overwrite=True)
backend = service.backend(name = "ibm_brisbane")
backend.num_qubits
#Do the Hello World Example on a 2 qubit bell state

#1: map the problem to circuits and operators
from qiskit import QuantumCircuit

qc = QuantumCircuit(2) 

qc.h(0)
qc.cx(0,1)

qc.draw(output = 'mpl')

# Map the problem to operators
from qiskit.quantum_info import Pauli

ZZ = Pauli('ZZ')
ZI = Pauli('ZI')
IZ = Pauli('IZ')
XX = Pauli('XX')
XI = Pauli('XI')
IX = Pauli('IX')

observables = [ZZ, ZI, IZ, XX, XI, IX]

#Step 2: Optimize operators

#Step 3: Execute on backend
from qiskit_aer.primitives import Estimator

estimator = Estimator()
job = estimator.run([qc]*len(observables), observables)
job.result()

#Step 4: Post processing/plotting

import matplotlib.pyplot as plt

data = ['ZZ','ZI', 'IZ', 'XX', 'XI', 'IX']
values = job.result().values
plt.plot(data, values, '-o')
plt.xlabel('observables')
plt.ylabel('Expectation Values')
def get_qc_for_n_qubit_GHZ_state(n):
    qc = QuantumCircuit(n)
    qc.h(0)
    for i in range(n-1):
        qc.cx(i, i+1)
    return qc
n = 100
qc = get_qc_for_n_qubit_GHZ_state(n)
qc.draw(output = 'mpl')
from qiskit.quantum_info import SparsePauliOp

operator_strings = ['Z'+ 'I' * i + 'Z' + 'I' * (n-2-i) for i in range(n-1)]
print(operator_strings)
print(len(operator_strings))

operators = [SparsePauliOp(operator_string) for operator_string in operator_strings]

from qiskit_ibm_runtime import QiskitRuntimeService
from qiskit.transpiler.preset_passmanagers import generate_preset_pass_manager

backend_name = "ibm_brisbane"
backend = QiskitRuntimeService().backend(backend_name)
pass_Manager = generate_preset_pass_manager(optimization_level=1, backend=backend)

qc_transpiled = pass_Manager.run(qc)
operator_transpiled_list = [op.apply_layout(qc_transpiled.layout) for op in operators]
from qiskit_ibm_runtime import EstimatorV2 as Estimator
from qiskit_ibm_runtime import EstimatorOptions

options = EstimatorOptions()
options.resilience_level = 1
options.dynamical_decoupling.enable = True
options.dynamical_decoupling.sequence_type = "XY4"

estimator = Estimator(backend, options=options)
job = estimator.run([(qc_transpiled, operator_transpiled_list)])
job_id = job.job_id()
print(job_id)
