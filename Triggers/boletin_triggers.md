# Boletín Triggers

## Todos estos ejercicios se hacen en la tabla de ejemplos SCOTT en Oracle DB.

## 1. Haz un trigger que solo permita a los vendedores tener comisiones.

```

create or replace trigger controlar_comisiones
before insert or update on emp
for each row
declare
	v_deptno dept.deptno%TYPE;
begin
	select deptno into v_deptno from dept where dname='SALES';

	if :new.job!='SALESMAN' or :new.deptno!=v_deptno and :new.comm is not null then
		raise_application_error(-20100,'Solo los vendedores pueden tener comisiones');
	end if;
end controlar_comisiones;
/

```

- Este trigger afecta a las insersiones o actualizaciones de la tabla "_EMP_", esto se hace **después** de la operación y afecta a cada fila.

- Se crea una variable que tenga el mismo tipo que el número de departamento de la tabla dept (dept.deptno // tabla.campo).

- En el cuerpo del trigger, lo primero es hacer un "_select_" que alamcene en la variable "_v_deptno_" el número de departamento cuyo nombre es "_SALES_"

- Lo siguiente, que activará el trigger es si el nuevo valor "**job**" insertado/actualizado es "**distinto**" a "*SALESMAN*" **ó** el nuevo "**departamento**" es distinto a la varibale y, además, la nueva comisión "**comm**" no es nula (Podría darse el caso de que no metiéramos un valor a este campo y los vendedores no pueden tener un valor nulo, al menos se debe insertar un 0 (?)). Si algo de esto no se cumple salta una excepción avisando del problema.


## 2. Registrar todas las operaciones sobre la tabla EMP de SCOTT en una tabla llamada AUDIT_EMP donde se guarde usuario, fecha, tipo de operación, fila afectada. (**POR PROBAR**)

```
create table audit_emp (
	usuario varchar(20),
	fecha	date,
	tipo_operacion varchar2(20)
);

```

- Crear una tabla que almacenará los datos que nos solicitan.

Procedimiento:

```
create or replace procedure insertar_auditoria (p_tipo varchar2(12))
is
begin
	insert into audit_emp values(user, sysdate, p_tipo);
	dbms_output.put_line('Añadido registro en Audit_Emp');
end insertar_auditoria;
/

```

- Creamos un procedimiento que será llamado por el trigger y que con un parámetro que le suministramos inserta los registros en la tabla que se ha creado previamente; mandando una salida de aviso porque se ha producido un registro. Creo que no sería necesario nada de esto, pero es para ir usando programación modular.

Trigger:

```
create or replace trigger AuditarEmp
after insert or update or delete on emp
declare
	v_operacion varchar2(12);
begin
	if inserting then
		v_operacion:='INSERTADO'
	elsif updating then
		v_operacion:='ACTUALIZADO'
	elsif deleting then
		v_operacion:='BORRADO'
	end if;
	insertar_auditoria(v_operacion);
end AuditarEmp;
/
```

- El trigger actua **antes** de que se inserte, actualice o borre un registro en la tabla de los empleados. Declaramos la variable que almacena el tipo de operacion que se está haciendo.

- En los condicionales definimos los tres posibles tipos de operación que se contemplan y este se almacena en la variable.

- Tras los condicionales, se llama al procedimiento dándole el registro de la variable para que pueda tener un correcto funcionamiento.


## 3.