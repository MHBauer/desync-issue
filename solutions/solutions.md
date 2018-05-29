
# Admission Controllers Always

Do all state changes in an admission controller before persisting them.

pros:
 - solves everything

cons:
 - loss of state/data if something gets in a bad state between acting and persistence. 
 - duplicates everything into the apiserver

# Queuing/Journaling/Append-Only Log

# 
