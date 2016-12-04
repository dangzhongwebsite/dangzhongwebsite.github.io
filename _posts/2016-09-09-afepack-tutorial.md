---
published: true
title: AFEPack Tutorial
layout: post
author:
category: tutorial 
tags: [AFEPack]
comments: true 
---

---


AFEPack �Ļ���ʹ�÷���������ͨ���������Ⲵ�ɷ� �̵ĳ��������彲�⡣��������������Ҫ���� �� $ \Omega = [0, 1]\times[0, 1] $ �������һ�����ϱ�ֵ����Ĳ��ɷ���

\[
-\Delta u = f, \qquad u |_{\partial \Omega} = u_b.
\]

��Ϊ����Ԫ���򣬵�һ���������Ҫ������� ���ʷ֣��õ���������ݡ� ��������ʹ��һ���򵥵Ķ�ά�����������ʷ� ���EasyMesh��ȥ��������ʷ֡� EasyMesh ��һ��һҳֽ��˵���飬������Լ�ȥ�Ķ��� ��EasyMesh ��Ҫ�û��ֹ�дһ������������������ļ��� Ϊ�����ļ�������ʹ�õ��ļ���Ϊ D.d�����������£�

4 # ����Ķ���ĸ��� #
0: 0.0 0.0 0.05 1
1: 1.0 0.0 0.05 1
2: 1.0 1.0 0.05 1
3: 0.0 1.0 0.05 1

4 # ����ı߽��ϱߵ����� #
0: 0 1 1
1: 1 2 1
2: 2 3 1
3: 3 4 1



����ǰ��һ���������������еĶ��㣬 ����4����Ȼ��ÿһ������һ���������Ϣ�� ������Ϊ

��������:    x����    y����    �ʷ��ܶ�h    ���ϱ�ʶ



����һ����������������ı߽��ϵıߵ����� ������4����Ȼ��ÿһ������һ��
�ߵ���Ϣ��������Ϊ

�ߵ����:    ��ʼ�������    �����������    ���ϱ�ʶ



������ļ������ļ��� D ��Ϊ���룬���� easymesh D�����ɻ���������ݣ� �洢��������ͬ���ļ��У� ��Ϊ D.n��D.s��D.e���洢�ķֱ��ǽڵ㡢�ߺ͵�Ԫ�� ��Ϣ��EasyMesh ��˵�����ж���Щ�ļ���ʽ����ϸ˵������ AFEPack Ҳ�ṩ�˶��������ݸ�ʽ���ļ�����Ľӿڡ�

�ڵõ������������Ժ����Ǽ����Կ�ʼ�Գ� �����˵�������ǵ����������ԭ�ĺ���ϸ�� ˵�����£�

/**
 * ������Щ������ͷ�ļ��������� AFEPack �ġ�
 */
#include <AFEPack/AMGSolver.h>        /// �����������������
#include <AFEPack/TemplateElement.h>  /// �ο���Ԫ
#include <AFEPack/FEMSpace.h>         /// ����Ԫ�ռ�
#include <AFEPack/Operator.h>         /// ����
#include <AFEPack/BilinearOperator.h> /// ˫��������
#include <AFEPack/Functional.h>       /// ����
#include <AFEPack/EasyMesh.h>         /// EasyMesh �ӿ�

#define DIM 2                /**< �����ά�� */
#define PI (4.0*atan(1.0))   /**< ���� \pi   */

/**
 * ��ȷ�� u �ı��ʽ
 *
 * @param p �����ϵĵ������: x = p[0], y = p[1]
 *
 * @return u(x, y)
 */
double _u_(const double * p)
{
  return sin(PI*p[0]) * sin(2*PI*p[1]);
}

/**
 *  �Ҷ���� f �ı��ʽ
 *
 * @param p �����ϵĵ������: x = p[0], y = p[1]
 *
 * @return f(x, y)
 */
double _f_(const double * p)
{
  return 5*PI*PI*_u_(p);
}

/**
 * �����������������е�ʱ������Ҫ����һ�������в�������Ϊ����������
 *         ���� D��
 *
 * @param argc �����в�������
 * @param argv �����в����б�
 *
 * @return ���� 0
 */
int main(int argc, char * argv[])
{
  /**
   * AFEPack �е� EasyMesh ���ܹ����� EasyMesh �����������ļ��Ľӿڣ�
   * ���������������� Mesh ���������ģ����Ե���һ������ʹ��
   */
  EasyMesh mesh;
  mesh.readData(argv[1]); /// �Ե�һ�������в���Ϊ�ļ�����������������

  /**
   * �������������Ϊ����Ԫ�ռ�׼���ο���Ԫ�����ݡ�AFEPack ���ο���Ԫ
   * �����ݷֳ�Ϊ�ĸ���ͬ����Ϣ��������
   *
   *   - �ο���Ԫ�ļ�����Ϣ
   *   - �ο���Ԫ�������еĵ�Ԫ������任
   *   - ��Ԫ�ϵ����ɶȷֲ�
   *   - ��Ԫ�ϵĻ�����
   *
   * ʹ���⼸����Ϣ������ϣ����Եõ��ܶ��ֲ�ͬ�Ĳο���Ԫ�����ĸ���Ϣ
   * �Ĺ���ʹ�����ĸ���ͬ���࣬��Ϊ TemplateGeometry, CoordTransform,
   * TemplateDOF �� BasisFunctionAdmin�����ĸ��඼���Դ������ļ��ж���
   * ��Ϣ������������Ľṹ�������ںܶೣ�õĵ�Ԫ����Щ�����ļ����Ѿ�
   * ׼������ AFEPack �� template ��Ŀ¼�¡���Щ�����ļ������ݸ�ʽ��
   * AFEPack ���ĵ�֮���бȽ���ϸ��˵����Ϊ���ܹ�ʹ�� AFEPack ˳������
   * ����Щ�����ļ���������Ҫ���û������� AFEPACK_TEMPLATE_PATH������
   * ���������������ļ��㣬 ������Ҫ����(���� bash)
   *
   * <pre>
   *   $ export AFEPACK_TEMPLATE_PATH=$AFEPACK_PATH/template/triangle
   * </pre>
   *
   * ���� AFEPACK_PATH ����Ϊ AFEPack �İ�װ·����
   */

  /**
   * �ο���Ԫ�ļ�����Ϣ������ģ����� DIM Ϊ�ο���Ԫ��ά�������������� 
   * �ڵ���������˵���� 2�� �ο���Ԫ�ļ�����Ϣ�а����ο���Ԫ�ļ��νṹ�� 
   * �Լ�������ֵ���ֵ�ʱ���һϵ�л��ֹ�ʽ����Ϣ��
   *
   */
  TemplateGeometry<DIM> triangle_template_geometry;
  triangle_template_geometry.readData("triangle.tmp_geo");

  /**
   * �Ӳο���Ԫ�������еĵ�Ԫ������任�������������ģ��������ֱ�Ϊ
   * �ο���Ԫ��ά���������е�Ԫ��ά�����������������ǲ�һ���ģ������
   * ��׼�����ε����������ε�����任����������ṩ�˽��ο���Ԫ�еĵ�
   * ��ʵ������Ԫ�еĵ��໥���б任�Լ��任���ſɱ�����ʽ�ļ��㷽����
   */
  CoordTransform<DIM,DIM> triangle_coord_transform;
  triangle_coord_transform.readData("triangle.crd_trs");

  /**
   * ���ɶȷֲ�ָ�����ڵ�Ԫ��ÿ�������������Ϸֲ������ɶȵĸ��������
   * ����Ҫ��֪���˲ο���Ԫ�ļ��ι��͵�����½��г�ʼ����
   */
  TemplateDOF<DIM> triangle_template_dof(triangle_template_geometry);
  triangle_template_dof.readData("triangle.1.tmp_dof");

  /**
   * �� BasisFunctionAdmin ��ʵ�Ϲ�����һ���������ÿ��������Ϊָ����
   * �������ļ����壬�ڲο���Ԫ�ϵĲ�ֵ�㣬һ���Ϊ"������ʶ��Э��"��
   * ��Ϣ���Լ�ÿ���������ĺ���ֵ�ͺ����ݶȵļ��㷽����
   */
  BasisFunctionAdmin<double,DIM,DIM> triangle_basis_function(triangle_template_dof);
  triangle_basis_function.readData("triangle.1.bas_fun");

  /**
   * ����������ĸ���Ϣ��ϵ��� TemplateElement �У��͵õ���һ��������
   * �ο���Ԫ������������Ԫ�ռ���Ҫһ�������һ��ο���Ԫ����������ֻ
   * ʹ��һ���ο���Ԫ��
   */
  std::vector<TemplateElement<double,DIM,DIM> > template_element(1);
  template_element[0].reinit(triangle_template_geometry,
                             triangle_template_dof,
                             triangle_coord_transform,
                             triangle_basis_function);

  /**
   * ʹ�����Ƕ���� EasyMesh �õ������񣬺͸ոմ����Ĳο���Ԫ������ʼ
   * ������Ԫ�ռ䡣
   */
  FEMSpace<double,DIM> fem_space(mesh, template_element);
        
  /**
   * ������ȡ�������еĵ�Ԫ�ĸ�����Ȼ��ݴ�Ϊ����Ԫ�ռ��еĵ�Ԫ������
   * ���еļ����嵥Ԫͬ�������Ŀռ䡣�˺�����ʹ������Ԫ�ռ��е�ÿ��
   * ��Ԫ�Ǹ��������е���Ӧ��ŵļ����嵥Ԫӳ���ϵ���Ųο���Ԫ�õ��ġ�
   */
  int n_element = mesh.n_geometry(DIM);
  fem_space.element().resize(n_element);
  for (int i = 0;i < n_element;i ++) {
    fem_space.element(i).reinit(fem_space,i,0);
  }

  /**
   * �������зֱ�������Ԫ�ռ��еĵ�Ԫ���ڲ����ݽṹ��Ȼ������������
   * �Ϸֲ����ɶ��Լ�Ϊ���е����ɶ�ȷ�����ϱ�ʶ���Դˣ���������Ԫ�ռ�
   * ���������ݾͶ�׼�����ˡ�
   */
  fem_space.buildElement();
  fem_space.buildDof();
  fem_space.buildDofBoundaryMark();

  /**
   * �����潨�����������Ԫ�ռ��ϣ����Ǽ���һ���նȾ��󣬾�����Ԫ��
   *
   * \f[
   *    a_{ij} = \int_\Omega \nabla \phi^i \cdot \nabla \phi^j dx
   * \f]
   *
   * �����ɵľ����������ǳ��þ�����˿��м���׼��������󡣽�����
   * ������������ʹ�����������Ԫ�ռ乹���������Ķ���Ȼ�����ü�
   * �����ʱ��ʹ�õ���ֵ���ֹ�ʽ�Ĵ������ȵĽ�������󣬵��ú���
   * build ���ɽ�������������
   */
  StiffMatrix<DIM,double> stiff_matrix(fem_space);
  stiff_matrix.algebricAccuracy() = 3;
  stiff_matrix.build();

  /**
   * ����Ԫ���� u_h �����ƽ��⺯�� u
   */
  FEMFunction<double,DIM> u_h(fem_space);

  /**
   * ���� f_h ���������Ҷ��� f ����ɢ����õ�����������Ԫ��Ϊ
   *
   * \f[
   *    f_i = \int_\Omega f \phi^i dx
   * \f]
   *
   * ���� Operator::L2Discretize �������������������ļ��㡣
   */
  Vector<double> f_h;
  Operator::L2Discretize(&_f_, fem_space, f_h, 3);

  /**
   * ���������ʹ���ϵ��ϱ�ֵ����������Ϊ���ϱ�ʶΪ 1 �����ɶ�ʹ�ú���
   * ���ʽ�������亯��ֵ���Ӷ�Ӧ��������Ҫ�ı�ֵ����������ͨ��ֱ����
   * �Ļ�õ�����ϵͳ�е�ϡ�������Ҷ�������������Ľ����
   */
  BoundaryFunction<double,DIM> boundary(BoundaryConditionInfo::DIRICHLET,
                                        1, &_u_);
  BoundaryConditionAdmin<double,DIM> boundary_admin(fem_space);
  boundary_admin.add(boundary);
  boundary_admin.apply(stiff_matrix, u_h, f_h);

  /**
   * ���� AFEPack �еĴ�������������������������ϵͳ
   */
  AMGSolver solver(stiff_matrix);
  solver.solve(u_h, f_h);      

  /**
   * ���������һ�������ļ��С���������ʹ���� Open Data Explorer ����
   * ���ļ���ʽ����������ݿ���ʹ�ø�����������ӻ���
   */
  u_h.writeOpenDXData("u.dx");

  /**
   * ������������о�ȷ�⣬���Ǽ���һ����ֵ��;�ȷ��� L^2 ��
   */
  double error = Functional::L2Error(u_h, FunctionFunction<double>(&_u_), 3);
  std::cerr << "\nL2 error = " << error << std::endl;

  return 0;
}



���������ʹ�� AFEPack ���г�����ƵĻ�����·����Ȼ���л�����û ���û�����Ĺ��ڼ��㷽������ݣ����Ǻܶ� ���ⶼ����ͨ�������������в�����޸ľ� 
����ɼ���f
