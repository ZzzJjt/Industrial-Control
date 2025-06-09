# model.py
from gekko import GEKKO

def build_model():
    m = GEKKO(remote=False)
    
    # Time horizon
    m.time = [0, 1, 2, 3, 4, 5]

    # Parameters
    T_sp = m.Param(value=520)  # °C
    P_min = 30  # bar
    T_max = 520  # °C

    # Control variable (input): Heat input rate (u)
    u = m.MV(value=0, lb=0, ub=1000)
    u.STATUS = 1

    # States: Temperature (T), Pressure (P)
    T = m.CV(value=300)  # °C
    T.STATUS = 1
    T.SPHI = T_max

    P = m.CV(value=10)   # bar
    P.STATUS = 1
    P.SPLO = P_min

    # Non-linear ODEs
    m.Equation(T.dt() == 0.1*u - 0.05*(T - 300))
    m.Equation(P.dt() == 0.08*u - 0.1*(P - 10))

    return m, u, T, P


# controller.py
from model import build_model

def run_nmpc():
    m, u, T, P = build_model()
    
    # Options
    m.options.IMODE = 6  # MPC
    m.options.CV_TYPE = 1
    m.options.NODES = 3
    m.solve(disp=False)

    # Print final predictions
    print(f"Final predicted T: {T[-1].value[0]:.2f} °C")
    print(f"Final predicted P: {P[-1].value[0]:.2f} bar")
    print(f"Optimal heat input profile: {u.value}")


# main.py
from controller import run_nmpc

if __name__ == '__main__':
    run_nmpc()
